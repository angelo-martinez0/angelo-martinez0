# Rexigon

### Lab Link
[WebVerse - Rexigon](https://dashboard.webverselabs-pro.com/academy/dom-based-xss/practice/challenge/rexigon?back=%2Facademy%2Fdom-based-xss%3Ftab%3Doverview)

---

### Overview
- Vulnerability Type: DOM-Based Cross-Site Scripting (DOM XSS)
- Affected Endpoint: /collection.php
- Vulnerable Parameter / Source: window.location.hash (e.g., #raptors)
- Security Controls: Content Security Policy (CSP) set to Report-Only mode (script-src 'self').

---

### Vulnerability Discovery
While auditing the application, checking the `View Source Page` and searching `script`; the endpoint `/collection.php` was identified as handling user-supplied data from the URL fragment `location.hash` identifier. Specifically, the application reads the anchor hash values such as `#raptors` to dynamically control page behavior or manipulate the Document Object Model (DOM).

### Source-to-Sink Analysis
Upon reviewing the page source code, the application uses hash values to jump straight to specific sets or content blocks:

```
<p class="rx-lede rx-lede--wide">The whole range on one page, grouped by set. Use the jump links to skip straight to a set, or share a link to one (the address ends in something like <span class="rx-mono">#raptors</span>) and it opens right there.</p>
```
If the application dynamically passes this unvalidated hash input into a DOM sink (such as inner HTML injection, unsafe image rendering via an onerror handler, or improper element creation), it results in execution of arbitrary JavaScript within the context of the victim's browser session.

### Content Security Policy (CSP) Context
#### During testing, it was observed that the application enforces a Content Security Policy (CSP) header configured in Report-Only mode (Content-Security-Policy-Report-Only).

- While an active CSP containing a strict script-src 'self' directive would normally block inline script execution, the Report-Only mode only logs violations rather than blocking them.

- Consequently, payloads designed around client-side DOM manipulation or specific event handlers (such as broken image fallbacks triggering onerror) can execute successfully without being actively mitigated by the browser.

---
#### Now we verified that the XSS vulnerability is in `collection.php#`

---
### Exploitation Workflow
`Input Injection:` We can now crafts a malicious URL pointing to the vulnerable endpoint with an injected payload in the hash fragment.

> https://2b8gl891-7197-rexigon-931e0challenges.webverselabs-pro.com/collection.php#
`<img src=x onerror=alert(document.domain)>`


`DOM Sink Processing:` The application reads the hash input client-side and injects it into the DOM, causing an intended asset (like an image) to fail to load.

`Execution:` The failure triggers an event handler (such as onerror), executing the arbitrary JavaScript payload entirely client-side.


*Reloading the page and...*
#### Flag is now Captured!
