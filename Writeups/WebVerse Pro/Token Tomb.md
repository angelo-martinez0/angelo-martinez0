# Token Tomb

### Lab Link
[WebVerse - Token Tomb](https://dashboard.webverselabs-pro.com/academy/jwt-attacks/practice/lab/token-tomb?back=%2Facademy%2Fjwt-attacks%3Ftab%3Doverview)

---

### Skills & Knowledge
- JSON Web Tokens (JWT)
- API Endpoint Enumeration
- Privilege Escalation

## Walkthrough & Exploitation
### Initial Access: 
Create a standard user account on the target platform and log in. Capture the session traffic using an interception proxy such as Caido or Burp Suite.

### Token Modification: 
Locate the issued JWT in the request headers. Decode the token and modify the header algorithm parameter to `alg: none` and the payload claim to `role: admin`.

This can be done using [JWT Tool](https://www.jwt.io/) or `Base64 Decode` in Caido.

### Privilege Escalation: 
Forward the modified request containing the forged token to escalate the current session's privileges to administrator level.

> {"alg":"none","typ":"JWT"}.{"username":"unknown","userid":1,"role":"admin","iat":1788429697,"exp":1788433297,"ext":1788433297}.

### Endpoint Discovery:

Navigate to the `Foreign Business Partners` section within the application interface or `/dashboard/partners` in the url.

The changes in the page will revert the role back to standard user, which needed to change the JWT to `admin` once again in the `/dashboard/partners`.

Review the response data returned from the Foreign Business Partners endpoint to uncover a hidden administrative API route: `/api/v1/admin/partnerdata`.

### Flag Retrieval: 
Navigate directly to the discovered API endpoint `/api/v1/admin/partnerdata` to access the restricted administrative data and retrieve the `Flag`.


