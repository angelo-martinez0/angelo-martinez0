# HookLink

### Lab Link
[WebVerse - HookLink](https://dashboard.webverselabs-pro.com/ranges/hooklink?back=%2Flearning-paths%2Fjunior-web-hacker)

---
In Caido HTTP History it can be noticed easily that every images request is showing `media.hooklink.local/photos/110/public/profile_1.jpg`. Proceed with IDOR by changing  the current photos to 1, changing to private, and removing the jpg

`Flag - Left in the open`
```
media.hooklink.local/photos/1/private/flag.txt
```

---

### Escalate Privilage

Change in Profile
Capture HTTP Response

`http://app.hooklink.local/me`

In the Request promote Subscriptions and Role to `gold` and `admin`

```
"subscription_tier":"gold",
"role":"admin"
```

`Admin` panel will be available

`System` page for `Flag - Backstage pass` 


`Reports` page inside Admin, clicking User with `Pending` statis would reveal the
`Flag - Everywhere you've been`


`Export` page in the manifest input will show proof that an RCE has reached
```
;ls;#
```
```
;ls ../;#
```
```
;cat ../flag.txt;#
```


---
### Account Take-over

After getting email from the Admin endpoint

`http://app.hooklink.local/forgot`

input the user with an account type: verified · gold

Capture HTTP Response

An endpoint can be seen in the Reponse

`http://app.hooklink.local/reset?token=6c05ed7f4a417cf97876c48ead6ae08233d1f5ba4e49051322a1bfe5fb3f51ac`

Change Password > Login > Profile
`Flag - Walk the guest list`

Message tab
`Flag - Return to sender`


