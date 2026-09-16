# Agent T

## Summary

- Platform: TryHackMe
- Category: Web Security / Vulnerability Research / Exploit Database / Remote Code Execution
- Status: Completed, as recorded in the source note
- Difficulty: Not recorded
- Report: Not created; see evidence limitations below.

This lab focused on identifying a vulnerable web server and using publicly available vulnerability information to gain access in a controlled lab environment.

The main objective was to scan the target, identify the exposed service, investigate the website, research the web server version for known vulnerabilities, use a matching public exploit, obtain a shell, and retrieve the flag.

## Environment

- Target: Authorized TryHackMe lab; target IP and testing date were not recorded.
- Wordlists or supporting files: Only those explicitly named in the workflow below; other values were not recorded.

### Tools

* Nmap
* Gobuster
* Web browser
* Exploit Database
* Public exploit script
* Terminal

## Enumeration

HTTP service on port 80; directory discovery and visible pages did not expose an entry point. The source then describes service-version research.

## Exploitation

### Recorded workflow

The following workflow is retained from the source note, including enumeration context and unsuccessful attempts. Commands and outcomes were documented by the author; they were not rerun during migration. Placeholder values are not recovered secrets.

1. Performed port scanning with Nmap to identify exposed services.
2. Found that only port `80` was open, indicating that the target was running a web service.
3. Used Gobuster to perform directory enumeration.
4. No useful hidden directories or files were discovered.
5. Visited the website through a web browser.
6. Investigated the web application manually, but nothing suspicious was found from the visible pages.
7. Shifted focus from the website content to the underlying web server.
8. Checked the web server information and researched whether the detected server version had known vulnerabilities.
9. Found a relevant vulnerability from Exploit Database.
10. Used the public exploit in the controlled TryHackMe lab environment.
11. Successfully exploited the vulnerable service and obtained a shell.
12. Searched the target system after gaining shell access.
13. Found and retrieved the flag.

## Root Cause

The source attributes shell access to a vulnerable web service, but does not record the product, version, vulnerability identifier, or exploit. A more specific root cause cannot be established.

## Remediation

The following defensive recommendations are retained from the source and are not evidence that remediation has been applied.

* Web servers and backend services must be kept up to date.
* Even when a website looks normal, the underlying server may still be vulnerable.
* Version disclosure can help attackers identify known vulnerabilities.
* Organisations should regularly patch exposed services.
* Public-facing services should be monitored for outdated or vulnerable software.
* Vulnerability scanning and patch management are important parts of web security.
* Public exploits can be dangerous when vulnerable software is exposed to the internet.

## Lessons Learned

### Key takeaways

* If directory enumeration and manual web investigation do not reveal anything useful, the next step is to investigate the underlying service version.
* Web server fingerprinting can reveal outdated or vulnerable software.
* Exploit Database is useful for researching known public exploits.
* A simple-looking website can still be vulnerable because of the server or framework behind it.
* Matching the exact service version to a known vulnerability is important before attempting exploitation.
* Public exploits should be tested only in legal and controlled environments.

### Reflection

This lab helped me understand that web exploitation is not always about hidden directories, login pages, or visible website functionality.

At first, port scanning showed that only port `80` was open. Directory scanning with Gobuster did not reveal useful results, and manual investigation of the website also did not show anything suspicious. Because of that, I changed my approach and started investigating the web server itself.

By researching the detected web server version, I found a known vulnerability listed in Exploit Database. After using the matching exploit, I was able to gain a shell and retrieve the flag.

This reinforced the importance of checking not only the web application but also the software and services running behind it.

### Skills practiced

* Port scanning
* Web service enumeration
* Directory enumeration
* Manual website investigation
* Web server fingerprinting
* Vulnerability research
* Searching Exploit Database
* Matching a service version to a known exploit
* Exploiting a vulnerable service in a lab environment
* Basic post-exploitation file discovery

## Evidence and Limitations

- Source: [Original learning note](https://github.com/Dongeun-Kang/cybersecurity/blob/3d27c869c37f6b8550ead2fa332fdb04b18dde9c/labs/THM_Agent_T.md) (immutable pre-migration revision).
- Documentation migration: 2026-09-16; this is not the testing date.
- Deferred: web-server product/version, exploit identifier, and request/output evidence are missing.
- The source does not contain raw session transcripts or screenshots. Omitted flags, credentials, private keys, and other sensitive values remain omitted.

## Original Disclaimer

This note is for educational purposes only.

All activity was performed in a legal, controlled TryHackMe lab environment.

Flags, credentials, and sensitive details are intentionally omitted.
