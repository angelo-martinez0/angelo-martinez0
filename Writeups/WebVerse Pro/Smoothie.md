# Smoothie

### Lab Link
[WebVerse - Smoothie](https://dashboard.webverselabs-pro.com/tracks/web-foundations/1/smoothie)

> Citrine Juice Co. is a one-bar operation in Boston's South End — six bar stools, a glass case of cold-pressed bottles, and a Saturday-morning regulars list taped to the side of the espresso machine. Margot opened it in 2019 and built the online-ordering site herself a year later. The login form was the last thing she touched before she stopped touching the code.

---

#### In the Lab Briefing there's a Hint already which mentioned `login`; which make the focus will be on there for now.


Doing some testing in Caido it can be seen in the Request that the `login` page `Content-Type` is recognised as a `JSON` database shape

```
POST /api/auth/login HTTP/1.1
Host: smoothie.local
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://smoothie.local/login
Content-Type: application/json
```
![alt text](/Writeups/images/Smoothieimage.png)

#### Since this is a JSON it will be a NoSQL Injection which specifically targeting a MongoDB-dialect datastore like NeDB

Which should be put in the email and password.

```
{"$ne: null}
```

> $ne: Stands for "not equal"  
null: Represents a null or empty value 

![alt text](/Writeups/images/Smoothieimage-1.png)

---

Which makes us login as Margot and the Response can now be view in the Browser; changing the directory from `/api/auth/login` to...

```
/account
```

![alt text](/Writeups/images/Smoothieimage-2.png)

Which now reveals the Flag

![alt text](/Writeups/images/Smoothieimage-3.png)