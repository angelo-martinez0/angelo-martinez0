# ActorSmactor

### Lab Link
[WebVerse - ActorSmactor](https://dashboard.webverselabs-pro.com/events/actorsmactor)
---
### Skills & Knowledge

- Recon
- Subversion Storage Architecture

---

#### Reconnaissance & Clue Analysis: 
The challenge briefing indicated that the website was `deployed by copying a local working directory` straight to the server, pointing directly toward potential version control artifacts.

### Content Discovery (ffuf): 
Running a directory brute-force scan with ffuf against the target uncovered several responses, notably highlighting a hidden `.svn` directory and its internal file at `/.svn/entries` with a 200 OK status.

```
ffuf -u https://8b3eb94a-5187-actorsmactor-f3720.events.webverselabs-pro.com/FUZZ -w /usr/share/wordlists/dirb/common.txt -e .php,.txt,.bak,.zip,.git -fc 404

```

### Identifying Version Control Artifacts: 
Accessing /.svn/entries exposed the Subversion metadata structure, revealing that tracking files such as config.php, index.php, and shows.php were present in the repository layout.

#### Purpose of /.svn/entries: 
>The /.svn/entries file is a flat metadata log used by Subversion to track file names, revisions, and statuses. It is not a directory containing the actual source code files.

#### Architecture of .svn/text-base/: 
>SVN stores pristine, readable backup copies of source files inside the hidden .svn/text-base/ directory (appending .svn-base to their names).

### Exploitation via .svn/text-base/:
Knowing that SVN stores pristine copies of tracked files inside `.svn/text-base/`, we leveraged curl to directly download the baseline source file without needing a full repository clone:
---

During reconnaissance, discovering an exposed /.svn/entries endpoint revealed that the developer had mistakenly uploaded their local Subversion working copy metadata directly to the production web server. This file acts as an internal ledger or manifest, listing versioned source files such as config.php, index.php, and shows.php that were tracked within the repository layout.

```
/.svn/entries
```

The server treats `.svn/entries` as a single metadata file, not a directory. Which will result to 404

`entries` only acts as a text list of filenames rather than a storage folder, you have to swap the path to point directly to the text-base directory where the actual source code file is kept.

```
/.svn/text-base/
```

---
### Other Approach

```
curl https://<target-id>.events.webverselabs-pro.com/.svn/text-base/config.php.svn-base -o config.php
```

### Flag Extraction:
Reviewing the contents of the downloaded config.php file revealed hardcoded API keys and credentials, successfully exposing the flag.