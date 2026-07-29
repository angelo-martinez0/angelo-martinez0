# Corridor

### Hints
- Listing of directory
- Developers notes html

---
#### Ridgeline Press is a three-person independent literary press in the Pacific Northwest. They publish a quarterly journal of fiction, essays, and poetry, and their site is a static-as-it-gets PHP app that loads each piece from a file on disk. No CMS, no database, just a folder of HTML fragments and one unlucky `readfile` call. Read the site. Find what shouldn't be there.
---
### Enumeration
- Reviewing the categories on the homepage reveals that navigating into any of the `Current Issue` pages exposes a URL structure that has `.php?slug=?`

 ![alt text](/Writeups/images/image1.png)

 - Which means it can direct us to the `directory` of the html

 - First thing to test is going back one directory using `../`

 ![alt text](/Writeups/images/image2.png)

---
#### Now a good practice is checking if there's a `notes.txt`

We can see the author `Mike` mention about `flag.txt` from his `home directory`. Now we know who and what the target is `Mike's Home directory`

![alt text](/Writeups/images/image4.png)

---
#### As we seen in the previous image we are inside /var/www/html. Next thing is going to the `root` directory by adding `../../../../` besides the `?=`
Should look like `?=../../../../`

 The ../../../../ Traversal: Each ../ moves the path up one level:

- First ../ moves out of the initial folder which is piece.php.

- Second ../ moves out of /var/www/html/.

- Third ../ moves out of /var/www/.

- Fourth ../ moves from /var/ to reach the Root Directory (/)

---
Now we can see the list inside the root. Notice in the last line we can see `/home/mike` remember he is the target, inside is the `flag.txt`

![alt text](/Writeups/images/image3.png)

---

### Capturing the Flag
#### Now we know where to go let's capture the flag.

```
../../../../home/mike/flag.txt
```

![alt text](/Writeups/images/image5.png)
