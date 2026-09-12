# Fawn

This is a HackTheBox Starting Point lab for practicing basic service enumeration, FTP access testing, and anonymous file retrieval.

## Reconnaissance

Started with **RustScan** and **Nmap** to identify open ports and services on the target.

```bash
target=10.129.1.14
rustscan --ulimit 5000 -a $target -- -sC -sV
```

The scan identified one open port:

```text
21/tcp open  ftp  vsftpd 3.0.3
```

Nmap also reported that anonymous FTP login was allowed:

```text
ftp-anon: Anonymous FTP login allowed
-rw-r--r--    1 0        0              32 Jun 04  2021 flag.txt
```

This showed that the FTP service exposed a readable `flag.txt` file to anonymous users.

---

## FTP Access

Connected to the target FTP service:

```bash
ftp $target
```

Logged in with the `anonymous` account:

```text
Name (10.129.1.14:kali): anonymous
331 Please specify the password.
230 Login successful.
```

The login succeeded, confirming anonymous access.

---

## File Enumeration

Listed the FTP directory:

```text
ftp> ls
```

Result:

```text
-rw-r--r--    1 0        0              32 Jun 04  2021 flag.txt
```

Downloaded the file:

```text
ftp> get flag.txt
```

The transfer completed successfully:

```text
226 Transfer complete.
32 bytes received
```

---

## Flag

Read the downloaded file:

```bash
cat flag.txt
```

Result:

```text
035db21c881520061c53e0536e44f815
```

---

## Attack Path

```text
RustScan / Nmap
        ↓
FTP Service Discovery
        ↓
Anonymous Login Identified
        ↓
FTP Directory Listing
        ↓
flag.txt Download
        ↓
Flag Retrieval
```

## Key Takeaways

* Anonymous FTP access can expose sensitive files without requiring credentials.
* Nmap's default scripts can quickly identify anonymous FTP access and readable files.
* FTP transmits data in plaintext and should not be exposed unless there is a clear operational need.
* Publicly accessible file services should be reviewed for unnecessary anonymous read or write permissions.
