# Nolic

### Lab Link
[WebVerse - Nolic](https://dashboard.webverselabs-pro.com/academy/content-discovery/practice/lab/nolic?back=%2Facademy%2Fcontent-discovery%2Fread%3Fsection%3Dbrute-forcing-with-a-wordlist%26path%3Djunior-web-pentester)

> Nolic is the long-form writing home of Wren Aldis, a Lisbon-based writer covering systems design, typography, and the slow web. The site runs on Wren's own self-hosted PHP stack — no analytics, no comments, no growth-hacking widgets. Just essays. Last week a draft post that Wren had not yet published appeared edited and live overnight under their own admin account. They have no idea how someone reached the admin panel. You've been brought in to find out — start at the public site as an anonymous reader and trace the path an attacker would have taken.

---

As mentioned in the modules, one of the best practice is checking to the `robots.txt` and finding unlinked or hidden directories, such as using ffuf, gobuster, dirbuster, or payloads in burpsuite or caido.

![alt text](/Writeups/images/imageNolic.png)

#### Upon checking the `/backups/` directory an `SQL Database` can be seen and inside it has a `Hash Password`

![alt text](/Writeups/images/imageNolic-1.png)

---

#### Using the `hash-identifier` tool we can identify what type of format is the hash

![alt text](/Writeups/images/imageNolic-2.png)

---

#### We now use `JohnTheRipper` tool to crack the hash password.
*Note put the hash password you found in a text file*

```
john --format=raw-sha256 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

![alt text](/Writeups/images/imageNolic-3.png)

> Now we can see that the hash password is now cracked which is sunshine

---

#### We create an essay, inside the body we will insert a PHP execution payload; listing what inside is in the directory;

![alt text](/Writeups/images/imageNolic-4.png)

then `View live`

![alt text](/Writeups/images/imageNolic-5.png)

#### Listing the root directory is the best practice

![alt text](/Writeups/images/imageNolic-6.png)

#### Inside the root there's an `entrypoint.sh` which is worthy to take a look.

![alt text](/Writeups/images/imageNolic-7.png)

#### And the exact location for the flag has been identified.

![alt text](/Writeups/images/imageNolic-9.png)

`{php}passthru('cat /home/wren/flag.txt');{/php}`

![alt text](/Writeups/images/imageNolic-8.png)