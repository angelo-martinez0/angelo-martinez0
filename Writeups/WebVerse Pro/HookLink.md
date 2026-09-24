# HookLink

### Lab Link
[WebVerse - HookLink](https://dashboard.webverselabs-pro.com/ranges/hooklink?back=%2Flearning-paths%2Fjunior-web-hacker)

---

## Summary

This lab is a chained exploitation exercise that demonstrates several critical web application flaws: insecure direct object reference (IDOR), privilege escalation through client-side trust, password reset abuse, command injection, and exposure of sensitive user data. Each stage builds on the previous one and reveals a new flag.

The overall flow is:

1. Abuse object access controls to read internal files
2. Escalate privileges by modifying the user object
3. Exploit admin functionality to access additional flags
4. Abuse password reset logic to take over an account
5. Exfiltrate sensitive location data from a profile

---

## 1. IDOR on profile images

While reviewing HTTP traffic in Caido, the profile image requests followed this pattern:

`media.hooklink.local/photos/110/public/profile_1.jpg`

This strongly suggested the application used a numeric photo ID and controlled access by a path segment such as `public` or `private`. By modifying the photo ID and switching the access path from `public` to `private`, we were able to access files that should not have been exposed.

Request:
```text
media.hooklink.local/photos/1/private/flag.txt
```

Flag:
```text
Flag - Left in the open
```

This confirmed the presence of an IDOR vulnerability and improper access control on stored files.

---

## 2. Privilege escalation

Open the account profile page and intercept the response from:

`http://app.hooklink.local/me`

The server returns a user object that includes account metadata. By modifying the JSON to set the user to a `gold` subscription and `admin` role, we can escalate privileges:

```json
{
  "subscription_tier": "gold",
  "role": "admin"
}
```

After submitting the modified response, the `Admin` panel becomes visible.

### System page
Inside the Admin area, the `System` page reveals the following:

```text
Flag - Backstage pass
```

### Reports page
Navigate to the `Reports` page and open a user whose status is `Pending`. This exposes:

```text
Flag - Everywhere you've been
```

### Export page / RCE via manifest input
On the `Export` page, the manifest input accepts unsanitized data. Injecting a command string triggers execution:

```text
;ls;#
```

To confirm the environment is reachable and directory traversal is possible, test:

```text
;ls ../;#
```

This reveals the underlying file structure, including the location of the flag file. To read it directly:

```text
;cat ../flag.txt;#
```

Flag:
```text
Flag - Everyone, all at once
```

This demonstrates command injection in the manifest processing flow and confirms the application is accepting untrusted input without proper validation.

---

## 3. Account takeover

After identifying the email address of an admin user, visit:

`http://app.hooklink.local/forgot`

Use an account with the type:

`verified · gold`

Capture the reset request and inspect the returned payload. The response includes a reset token, similar to:

```text
http://app.hooklink.local/reset?token=6c05ed7f4a417cf97876c48ead6ae08233d1f5ba4e49051322a1bfe5fb3f51ac
```

Using this token, we can reset the password and log in as the admin user. After authenticating, open the user profile.

Flag:
```text
Flag - Walk the guest list
```

Then open the `Messages` tab to access additional sensitive information:

```text
Flag - Return to sender
```

This shows that password-reset tokens were exposed or insufficiently protected, enabling account takeover.

---

## 4. Crossed paths

Using the Admin `/users` endpoint, locate Maya’s email address and reset her password. Once logged in as Maya, inspect her profile. Her exact coordinates are exposed either in the response body or via the browser developer tools under the Network tab.

The location can then be queried using the nearby endpoint:

```text
/api/nearby?lat=25.795319&lng=-80.134167
```

This reveals:

```text
Flag - Crossed paths
```

This step demonstrates the exposure of sensitive personal information in a user profile and highlights the danger of returning private metadata without adequate access restrictions.

---

## Final Notes

This lab contains multiple vulnerabilities that can be chained together to achieve full compromise:

- IDOR on file access
- Broken access control in profile/account data
- Privilege escalation via user object manipulation
- Insecure command execution in export functionality
- Weak password reset flow and exposed reset tokens
- Sensitive data exposure in user profile fields
