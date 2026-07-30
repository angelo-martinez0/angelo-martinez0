# Flower

### Lab Link
[WebVerse - Flower](https://dashboard.webverselabs-pro.com/tracks/web-foundations/1/flower)

### Hint
SQL injection

---

> Flower Haven is a small neighborhood florist. They just went live with an online store over the weekend, built quickly, launched quickly, with nothing reviewed before it went up. The site is taking real orders now. Browse the catalog, try out the features, and pay attention to how the shop responds to what you give it.

---
### Analysis
First thing is we check the `url`, `request`, and `response` of all the category in the page if something can be exploited

![alt text](/Writeups/images/image6.png)

An interesting part is the `Home` page, as it contains a `title` `description` `price`. This is one of the common part to do an SQL injection so we could see the content of the database.

![alt text](/Writeups/images/image7.png)
### Enumeration



```

```