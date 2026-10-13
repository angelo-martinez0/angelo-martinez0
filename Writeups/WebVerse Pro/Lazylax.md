# Lazylax

### Lab Link
[WebVerse - Lazylax](https://dashboard.webverselabs-pro.com/events/lazylax)

---

### Skills & Knowledge
- API Reconnaissance (JavaScript Source Review)
- Directory/Endpoint Fuzzing
- NoSQL Database Misconfiguration (CouchDB)
- Broken Access Control / Sensitive Data Exposure

---

## Vulnerability Overview

### Vulnerability Type: Broken Access Control via Exposed CouchDB REST API leading to Sensitive Data Exposure.

> *Target: `/db/` proxy endpoint backing the photo feed.*

---

### Initial Reconnaissance

#### The application ("Lazylax") renders a photo feed of scans/photos posted by users, with no obvious interactive attack surface on the page itself.

Viewed the page source and located the feed's client-side script:
```
view-source:https://<target>/static/app.js
```

The source revealed the feed is populated with a raw client-side fetch:
```js
/* Lazylax feed. Reads the photo documents straight from CouchDB through the /db/
 * proxy and renders the grid. */
fetch('/db/photos/_all_docs?include_docs=true')
  .then(function (r) { return r.json(); })
  .then(function (d) {
    ...
    rows.forEach(function (row) {
      if (row.doc && row.doc.title) { feed.appendChild(tile(row.doc)); }
    });
  })
```

> Critical detail: the comment and code confirm `/db/` is a direct proxy into a CouchDB instance, not an app-defined API. The frontend only **renders** documents that have a `title` field — it does not restrict what the backend actually returns. `_all_docs` and `include_docs=true` are standard CouchDB REST conventions, not custom routes, meaning the full CouchDB HTTP API was likely reachable through the same `/db/` path.

---

### Enumerating the Database

Since `/db/` forwards requests straight to CouchDB, the proxy path was fuzzed for additional, non-public databases beyond `photos`:

```
ffuf -u https://target.com/db/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt -fc 404
```

The fuzzing turned up `_users`, a CouchDB system database, reachable through the same unauthenticated proxy — and critically, nothing in the frontend's `app.js` ever queries this database, meaning its contents were never meant to be reachable by a normal client.

Confirming `photos` itself held nothing extra:
```
GET /db/photos/_all_docs
```
Returned exactly 6 document stubs, matching the 6 tiles already visible in the rendered feed — ruling out hidden docs in `photos` and pointing the investigation squarely at `_users`.

---

### Sensitive Data Exposure & Flag Retrieval

Applied the same CouchDB convention used by the app itself (`_all_docs?include_docs=true`) against the fuzzed-out database:

```
GET /db/_users/_all_docs?include_docs=true
```

> Because `/db/` performs no path- or database-level access control, any database on the CouchDB instance — not just the one the application intended to expose — was readable with the same unauthenticated request pattern.

The response returned the full document bodies for `_users`, normally never exposed to the client since the frontend only ever requests `/db/photos/...`. The flag was contained directly in that response:

```
WEBVERSE{...}
```

> Root cause: the `/db/` reverse proxy forwarded requests to CouchDB with no per-database allowlist and no authentication, trusting the frontend's own restraint (`if (row.doc && row.doc.title)`) as if it were a security boundary. Any client capable of fuzzing or making raw HTTP requests could bypass that entirely using documented CouchDB API calls.

---

### Remediation
- Restrict the `/db/` proxy to the specific database and endpoints the frontend needs (`photos/_all_docs`, `photos/_design/*`) rather than forwarding the full CouchDB API.
- Require authentication on the proxy, independent of CouchDB's own `_security` settings.
- Never rely on client-side filtering (`if (row.doc && row.doc.title)`) as an access control mechanism — it only hides data from the rendered UI, not from direct API access.
