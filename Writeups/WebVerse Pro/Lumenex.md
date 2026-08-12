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
Since { "$type": "string" } successfully returned the 7 standard public products, it proves that the endpoint is executing queries correctly against the products collection, but those 7 items are all that exist there.

To double check we use regular expression, to bypass exact matching restrictions and match every single document in the database collection that contains a category field
```
"category": {
"$type": "string"￼  ￼}
```

```
ffuf -u https://38ff353e-5187-lumenex-282ec.challenges.webverselabs-pro.com/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt -fc 404
```