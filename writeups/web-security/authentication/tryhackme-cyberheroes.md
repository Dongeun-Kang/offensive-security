# CyberHeroes

## Summary

- Platform: TryHackMe
- Category: Web Security / Reconnaissance / Source Code Review / Weak Credential Exposure
- Status: Completed, as recorded in the source note
- Difficulty: Not recorded
- Report: [Assessment report](../../../reports/tryhackme/cyberheroes-pentest-report.md)

This lab focused on basic web application enumeration and identifying exposed credentials through source code review.

The main objective was to scan the target, discover available web resources, investigate the website manually, identify exposed login information, understand how the password was modified, and use the corrected credential to authenticate successfully.

## Environment

- Target: Authorized TryHackMe lab; target IP and testing date were not recorded.
- Wordlists or supporting files: Only those explicitly named in the workflow below; other values were not recorded.

### Tools

* Nmap
* Gobuster
* Web browser
* View Source
* Linux terminal
* `echo`
* `rev`

## Enumeration

Port 80 exposed a web application. Client-accessible source revealed login information associated with `login.php` and a `ReverseString()` transformation.

## Exploitation

### Recorded workflow

The following workflow is retained from the source note, including enumeration context and unsuccessful attempts. Commands and outcomes were documented by the author; they were not rerun during migration. Placeholder values are not recovered secrets.

1. Performed port scanning with Nmap to identify exposed services.

2. Found that port `80` was open, indicating that a web service was running.

3. Used Gobuster to perform directory enumeration and discover hidden web paths.

4. Visited the website through a web browser.

5. Investigated the website manually and reviewed available pages.

6. Used View Source to inspect the HTML and client-side code.

7. Found credentials related to `login.php`.

8. Identified that the password had been reversed using a `ReverseString()` function.

9. Reversed the password again using the terminal command:

   ```bash
   echo "value" | rev
   ```

10. Initially considered using Python to reverse the string, but later realised that the Linux `rev` command was a simpler and faster option.

11. Used the corrected credential to log in successfully.

12. Retrieved the lab flag.

## Root Cause

Authentication material was sent to the browser. Reversing the password string was reversible obfuscation, and the recovered credential reportedly authenticated successfully.

## Remediation

The following defensive recommendations are retained from the source and are not evidence that remediation has been applied.

* Sensitive credentials should never be exposed in source code.
* Client-side code should not contain secrets, passwords, or authentication logic.
* Reversing a password or applying simple string manipulation is not encryption.
* Authentication secrets must be stored securely on the server side.
* Web directories and files should be reviewed before deployment to avoid accidental exposure.
* Developers should assume that anything sent to the browser can be inspected by users.
* Proper credential management and secure coding practices are required to prevent easy account compromise.

## Lessons Learned

### Key takeaways

* Port scanning is useful for identifying which services are exposed on a target.
* Directory enumeration can reveal hidden or unlinked web pages.
* Source code review can expose sensitive information if developers leave credentials or logic in client-accessible files.
* Simple encoding or string manipulation does not protect passwords.
* Linux command-line tools can often solve simple tasks faster than writing a custom script.
* Exposed credentials can directly lead to unauthorised access if proper security controls are not applied.

### Reflection

This lab reinforced the importance of structured web enumeration. I practiced starting from service discovery, moving into directory scanning, and then manually reviewing the website for useful information.

I also learned that simple password transformation, such as reversing a string, is not a secure protection method. If the logic is visible in the source code, an attacker can understand and reverse the process easily.

This lab also helped me realise that using the simplest available tool is often better than overcomplicating the task. Although Python could reverse the string, the `echo "value" | rev` command was quicker and more efficient.

### Skills practiced

* Port scanning
* Web directory enumeration
* Website investigation
* Reviewing page source code
* Identifying exposed credentials
* Understanding simple string manipulation
* Using Linux terminal commands to reverse a string
* Authenticating to a web application in a controlled lab environment

## Evidence and Limitations

- Source: [Original learning note](https://github.com/Dongeun-Kang/cybersecurity/blob/3d27c869c37f6b8550ead2fa332fdb04b18dde9c/labs/THM_CyberHeroes.md) (immutable pre-migration revision).
- Documentation migration: 2026-09-16; this is not the testing date.
- Report: source identifies the exposed login material, reversible transformation, and successful authentication.
- The source does not contain raw session transcripts or screenshots. Omitted flags, credentials, private keys, and other sensitive values remain omitted.

## Original Disclaimer

This note is for educational purposes only.

All activity was performed in a legal, controlled TryHackMe lab environment.

Flags, credentials, and sensitive details are intentionally omitted.
