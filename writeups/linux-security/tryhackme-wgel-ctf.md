# Wgel CTF

## Summary

- Platform: TryHackMe
- Category: Linux Privilege Escalation / Web Enumeration / SSH Key Discovery / Sudo Misconfiguration
- Status: Completed, as recorded in the source note
- Difficulty: Not recorded
- Report: [Assessment report](../../reports/tryhackme/wgel-ctf-pentest-report.md)

This lab focused on web enumeration, identifying exposed sensitive files, using an SSH private key for initial access, and abusing a misconfigured privileged command to read the root flag.

The main objective was to scan the target, identify open services, investigate the website, discover an exposed `id_rsa` file, use it to log in through SSH, and escalate privileges by abusing `wget` permissions.

## Environment

- Target: Authorized TryHackMe lab; target IP and testing date were not recorded.
- Wordlists or supporting files: Only those explicitly named in the workflow below; other values were not recorded.

### Tools

* Nmap
* Gobuster
* Web browser
* View-source
* SSH
* Wget
* Terminal

## Enumeration

Ports 80 and 22 were open. Page source exposed a username, directory discovery found `id_rsa`, and post-login `sudo -l` allowed elevated `wget`.

## Exploitation

### Recorded workflow

The following workflow is retained from the source note, including enumeration context and unsuccessful attempts. Commands and outcomes were documented by the author; they were not rerun during migration. Placeholder values are not recovered secrets.

1. Performed a full port scan with Nmap to identify open ports on the target.
2. Ran a detailed Nmap scan against the discovered open ports.
3. Found that ports `80` and `22` were open.
4. Visited the web service running on port `80`.
5. Investigated the website manually.
6. Reviewed the page source and eventually found a suspicious username.
7. Missed the username at first, but found it later after revisiting the source code.
8. Used Gobuster to perform directory discovery against the web server.
9. Discovered an exposed `id_rsa` file.
10. Downloaded the private key and attempted SSH login using the discovered username.
11. Successfully logged in to the target through SSH.
12. Retrieved the user flag.
13. Ran `sudo -l` to check what commands the user could run with elevated privileges.
14. Found that the user was allowed to run `wget` with elevated privileges.
15. Used `wget` to read `/root/root_flag.txt`.
16. Successfully retrieved the root flag.

## Root Cause

An exposed SSH private key enabled login, and excessive sudo permission for `wget` allowed access to a root-owned file. The methodology supports a sudo misconfiguration, not a demonstrated SUID issue.

## Remediation

The following defensive recommendations are retained from the source and are not evidence that remediation has been applied.

* Sensitive files such as SSH private keys should never be stored in publicly accessible web directories.
* Web source code should be reviewed carefully to avoid exposing usernames, comments, paths, or other useful information.
* Directory listing and exposed files can create serious security risks.
* SSH private keys must be protected with strict file permissions and should not be reused carelessly.
* Sudo permissions should follow the principle of least privilege.
* Allowing users to run file retrieval tools such as `wget` with elevated privileges can lead to unauthorized access to restricted files.
* Regular audits of web directories and sudo configurations are important for preventing privilege escalation.

## Lessons Learned

### Key takeaways

* Page source code can contain useful information that is not visible on the webpage.
* Important clues can be missed during the first manual inspection, so revisiting earlier findings is valuable.
* Directory enumeration can reveal sensitive files accidentally exposed on a web server.
* Exposed SSH private keys can lead directly to initial access if matched with a valid username.
* `sudo -l` is an important privilege escalation check after gaining a shell.
* Misconfigured sudo permissions can allow a normal user to access restricted root files.
* Tools like `wget` can become dangerous when allowed to run with elevated privileges.

### Reflection

This lab helped me understand the importance of carefully checking both the visible website and the source code behind it.

At first, the Nmap scan showed that only ports `80` and `22` were open. Since port `80` was available, I investigated the website manually and also used Gobuster for directory enumeration. During the first inspection, I missed a suspicious username inside the page source. After coming back to the source code later, I found the username and realised it was important for SSH access.

Gobuster helped reveal an exposed `id_rsa` file. Since an SSH service was running on port `22`, I used the discovered username together with the private key to log in successfully. After gaining access, I retrieved the user flag.

For privilege escalation, I checked sudo permissions using `sudo -l` and found that `wget` could be used with elevated privileges. By abusing this misconfiguration, I was able to read the root flag from `/root/root_flag.txt`.

This lab reinforced that small pieces of information, such as usernames and exposed private keys, can become critical when combined together.

### Skills practiced

* Full port scanning
* Service enumeration
* Web application investigation
* Reviewing page source code
* Directory enumeration
* Identifying exposed SSH private keys
* SSH login using `id_rsa`
* Linux privilege escalation enumeration
* Checking sudo permissions with `sudo -l`
* Abusing misconfigured privileged binaries
* Reading restricted files through command misuse

## Evidence and Limitations

- Source: [Original learning note](https://github.com/Dongeun-Kang/cybersecurity/blob/3d27c869c37f6b8550ead2fa332fdb04b18dde9c/labs/THM_Wgel%20CTF.md) (immutable pre-migration revision).
- Documentation migration: 2026-09-16; this is not the testing date.
- Report: exposed key, successful SSH authentication, privileged wget, and `/root/root_flag.txt` access are explicitly recorded.
- The source does not contain raw session transcripts or screenshots. Omitted flags, credentials, private keys, and other sensitive values remain omitted.
- Classification corrected from the source's SUID label to sudo based on its `sudo -l` workflow. Reading the root flag does not establish an interactive root shell.

## Original Disclaimer

This note is for educational purposes only.

All activity was performed in a legal, controlled TryHackMe lab environment.

Flags, credentials, and sensitive details are intentionally omitted.
