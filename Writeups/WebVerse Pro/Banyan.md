# Banyan

### Lab Link
[WebVerse - Banyan](https://dashboard.webverselabs-pro.com/academy/reflected-xss/practice/challenge/banyan?back=%2Facademy%2Freflected-xss%3Ftab%3Doverview)

---

### Skills & Knowledge
- Cross-Site Scripting, Reflected

---

Testing the XSS with special character, see if how the input is being reflected and handled by the server in the response pane.

`Vulnerable:` If the response shows the raw tags—meaning it renders as 
```
<blockquote><script>alert(1)</script></blockquote>
```
—the application is vulnerable because it is not encoding special characters.

`Secure:` If the response encodes the characters into safe HTML entities—meaning it renders as 
```
<blockquote>&lt;script&gt;alert(1)&lt;/script&gt;</blockquote>
```
the application is safe against basic injection in that context; `but not if we encode the special characters`.

> Test using characters only
![alt text](/Writeups/images/Banyanimage.png)


> Test using special character for XSS
![alt text](/Writeups/images/Banyanimage-1.png)

Notice that the ```<script``` has been filtered out

From what we tackle we could use different approachs
- Changing the case
```<sCriPt>```
- Replacing the `space` to `/`: `<svg/onload=alert(document.domain)>`
- Encode or double encode the special characters `%253Cscript%253E`
- Nesting `<scr<script>ipt>alert(1)</script>`

---

### Analysis of Application Defenses:

- `Double-Encoding Behavior:` The server performs a single decode pass during transit, outputting literal URL strings like `%3Cscript%3E` into the HTML structure rather than evaluating them as active tags.

- `Input Sanitization:` Direct `<script` declarations are actively filtered out by the backend.

- `Whitespace & Filter Evasion:` Replacing spaces with a /, tab, or newline bypasses space-sensitive WAFs while keeping the HTML parser valid

### Successful Payload: SVG Vector

To successfully execute code in the application, I injected a compact SVG-based payload; as mentioned replacing the space to `/`:

```html
<svg/onload=alert(document.domain)>
```

![alt text](/Writeups/images/Banyanimage-2.png)

XSS successful, we now Captured the Flag

![alt text](/Writeups/images/Banyanimage-3.png)


