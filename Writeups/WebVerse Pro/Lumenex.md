# Lumenex

### Lab Link
[WebVerse - Lumenex](https://dashboard.webverselabs-pro.com/academy/nosql-injection/practice/challenge/lumenex?back=%2Facademy%2Fnosql-injection%3Ftab%3Doverview)

---

### Skills & Knowledge
- NoSQL - Filter and search injection
- ffuf

---

Checking the product filter search

```
{
  "category": { "$type": "string" }
}
```
Since `{ "$type": "string" }` successfully returned the 7 standard public products, it proves that the endpoint is executing queries correctly against the products collection, but those 7 items are all that exist there.

---

Now checking if there's a hidden/unlink path

```
ffuf -u https://38ff353e-5187-lumenex-282ec.challenges.webverselabs-pro.com/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt -fc 404
```

As per checking using ffuf `/register` has been revealed, so creating an account is a must to check further.

Now we continue to put NoSQL Injection on the same path but in the created accounted.

> *Note: As the lesson notes state, some endpoints stack two or three default filters, and overriding just one won't work—you have to defeat them all in the same request.*

Since the standard public products that are shown contains some filter it might be worth it to add some of this to reveal the `unreleased` products.

![alt text](/Writeups/images/Lumeneximage1.png)

`status` and `visibility` looks interesting to check.

```
"status": { "$ne": null },
  "visibility": { "$ne": null },
  "released": { "$ne": null }
```

Using three filters return empty response, the techique should test one by one to figure out which guard is blocking the query.

As notice `"status": { "$ne": null }` only return the same response where it shows all 7 products

`"visibility": { "$ne": null }` return a new response, returns 8 products

`"released": { "$ne": null }` also returns 8 products

Which means combining the 2 might find something worthy

```
"visibility": { "$ne": null },"status": { "$exists": true }
```

Flag has been captured!

![alt text](/Writeups/images/Lumeneximage.png)