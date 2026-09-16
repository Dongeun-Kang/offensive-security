# Simple CTF

## Summary

- Platform: TryHackMe
- Category: Web Security / CMS Exploitation / Vulnerability Research / Credential Cracking / Privilege Escalation
- Status: Completed, as recorded in the source note
- Difficulty: Not recorded
- Report: [Assessment report](../../reports/tryhackme/simple-ctf-pentest-report.md)

This lab focused on enumerating a web application, identifying a vulnerable CMS, using a public exploit to obtain credentials, cracking a hashed password, and escalating privileges through a misconfigured sudo permission.

The main objective was to scan the target, discover the web application, identify the CMS and its version, research known vulnerabilities, exploit the vulnerable service to obtain credentials, gain SSH access, and escalate privileges to root.

## Environment

- Target: Authorized TryHackMe lab; target IP and testing date were not recorded.
- Wordlists or supporting files: Only those explicitly named in the workflow below; other values were not recorded.

### Tools

* Nmap
* Gobuster
* Web browser
* SearchSploit
* Public exploit script
* Hashcat
* SSH
* sudo
* Vim
* Terminal

## Enumeration

Enumeration identified CMS Made Simple. After credential recovery and SSH access, `sudo -l` reportedly allowed `/usr/bin/vim`.

## Exploitation

### Recorded workflow

The following workflow is retained from the source note, including enumeration context and unsuccessful attempts. Commands and outcomes were documented by the author; they were not rerun during migration. Placeholder values are not recovered secrets.

1. Performed a full port scan with Nmap to identify all open ports on the target.
2. Ran a detailed Nmap scan against the discovered open ports.
3. Found a suspicious web application running on the target.
4. Visited the website through a web browser and began manual investigation.
5. Used Gobuster to perform directory discovery on the web service.
6. Discovered a service running CMS Made Simple.
7. Identified the CMS version and searched for known vulnerabilities using SearchSploit.
8. Found a relevant public exploit matching the CMS Made Simple version.
9. Ran the exploit in the controlled TryHackMe lab environment.
10. Successfully extracted credentials from the vulnerable CMS.
11. Found that the password was stored as a hash rather than plaintext.
12. Used Hashcat to crack the password hash.
13. Logged in to the target machine through SSH using the recovered credentials.
14. Retrieved the `user.txt` flag.
15. Ran `sudo -l` to check which commands the user could execute with sudo privileges.
16. Found that `/usr/bin/vim` was allowed to run as sudo.
17. Used Vim’s sudo permission to escalate privileges.
18. Gained root access and retrieved the `root.txt` flag.

## Root Cause

The recorded local privilege escalation resulted from sudo permission for an editor capable of invoking commands. The separate CMS weakness cannot be assigned a version, CVE, or injection type from this source.

## Remediation

The following defensive recommendations are retained from the source and are not evidence that remediation has been applied.

* Public-facing CMS platforms must be kept up to date.
* Outdated CMS versions can expose systems to known public exploits.
* Software version disclosure can help attackers find matching vulnerabilities.
* Credentials should never be stored insecurely or exposed through vulnerable applications.
* Password hashes should be protected and should use strong hashing algorithms with proper salting.
* Weak passwords can still be recovered if attackers obtain the hash.
* SSH access should be protected with strong passwords, key-based authentication, and limited exposure.
* Sudo permissions should follow the principle of least privilege.
* Dangerous binaries such as Vim should not be allowed to run with sudo unless absolutely necessary.
* Regular privilege audits can help identify misconfigurations before attackers abuse them.
* Vulnerability management, patching, and secure privilege configuration are all necessary for reducing attack risk.

## Lessons Learned

### Key takeaways

* Full port scanning is important because relying only on default ports can miss exposed services.
* Detailed service enumeration helps identify suspicious applications and software versions.
* Directory enumeration can reveal hidden web paths that expose useful information.
* CMS version information can be used to search for known public vulnerabilities.
* SearchSploit is useful for quickly checking whether a known exploit exists for a specific software version.
* Public exploits may reveal credentials, but those credentials may still require additional work such as hash cracking.
* Hash cracking is an important step when leaked or extracted credentials are not in plaintext.
* SSH access with valid credentials can provide an initial foothold.
* `sudo -l` is a key Linux privilege escalation check.
* Misconfigured sudo permissions can allow privilege escalation if dangerous binaries are allowed to run as root.
* Vim can be abused for privilege escalation when it is allowed to run with sudo privileges.

### Reflection

This lab helped me understand the full attack flow from enumeration to privilege escalation.

At first, I performed a full Nmap port scan to identify exposed services. After running a more detailed scan on the open ports, I found a suspicious web application. I then used Gobuster to enumerate directories and discovered that the application was running CMS Made Simple.

Instead of guessing randomly, I checked the CMS version and used SearchSploit to look for known vulnerabilities. A matching public exploit was available, and using it allowed me to obtain credentials from the vulnerable service.

However, the password was not directly usable because it was hashed. I cracked the hash with Hashcat and then used the recovered credentials to log in through SSH. After gaining access, I retrieved the user flag and moved on to privilege escalation.

Running `sudo -l` showed that the user could run `/usr/bin/vim` with sudo privileges. Since Vim can execute system commands, this sudo permission could be abused to gain root access. After escalating privileges, I retrieved the root flag.

This reinforced that exploitation does not stop after gaining initial access. Post-exploitation enumeration and privilege escalation checks are necessary to fully compromise a lab machine.

### Skills practiced

* Full port scanning
* Service enumeration
* Web application investigation
* Directory enumeration
* CMS fingerprinting
* Vulnerability research
* Searching public exploits with SearchSploit
* Exploiting a vulnerable CMS in a lab environment
* Hash cracking
* SSH login with discovered credentials
* Linux post-exploitation enumeration
* Checking sudo permissions
* Privilege escalation using a sudo-allowed binary

## Evidence and Limitations

- Source: [Original learning note](https://github.com/Dongeun-Kang/cybersecurity/blob/3d27c869c37f6b8550ead2fa332fdb04b18dde9c/labs/THM_Simple%20CTF.md) (immutable pre-migration revision).
- Documentation migration: 2026-09-16; this is not the testing date.
- Report limited to the identified Vim sudo misconfiguration; the unspecified CMS exploit remains attack-chain context.
- The source does not contain raw session transcripts or screenshots. Omitted flags, credentials, private keys, and other sensitive values remain omitted.

## Original Disclaimer

This note is for educational purposes only.

All activity was performed in a legal, controlled TryHackMe lab environment.

Flags, credentials, hashes, and sensitive details are intentionally omitted.
