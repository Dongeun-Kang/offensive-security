# Recruit

## Summary

- Platform: TryHackMe
- Category: Web Security / Enumeration / Local File Inclusion / SQL Injection
- Status: Completed, as recorded in the source note
- Difficulty: Not recorded
- Report: Not created; see evidence limitations below.

This lab focused on web application enumeration, identifying exposed directories, discovering user-related information, and chaining multiple vulnerabilities to access both user-level and admin-level flags.

The main objective was to practice a structured web testing workflow and understand how information disclosure, local file access, and SQL injection can be chained during a lab assessment.

## Environment

- Target: Authorized TryHackMe lab; target IP and testing date were not recorded.
- Wordlists or supporting files: Only those explicitly named in the workflow below; other values were not recorded.

### Tools

- Nmap
- Gobuster
- Hydra
- Burp Suite
- SQLmap
- Web browser
- Linux CLI

## Enumeration

Directory discovery disclosed a username and local-file clues. Initial Hydra testing failed, after which a PHP file-access function and a captured database request were investigated.

## Exploitation

### Recorded workflow

The following workflow is retained from the source note, including enumeration context and unsuccessful attempts. Commands and outcomes were documented by the author; they were not rerun during migration. Placeholder values are not recovered secrets.

1. Performed an Nmap scan and identified an open web service.
2. Visited the website to understand the application.
3. Used Gobuster to discover hidden directories.
4. Found a username from one of the discovered directories.
5. Tested the discovered username with Hydra, but the credentials did not provide access.
6. Continued enumeration and found a PHP file that could access local files.
7. Identified a local file from the discovered directories.
8. Used the PHP file access behaviour to read the local file.
9. Found a valid user password and obtained the user-level flag.
10. Identified that the web application used a SQL database.
11. Captured an HTTP request with Burp Suite.
12. Used SQLmap with the captured request to test for SQL injection.
13. Retrieved admin credentials from the vulnerable database.
14. Used the admin credentials to obtain the admin-level flag.

## Root Cause

The source describes unrestricted local-file access and SQL injection exposing credentials. It does not include the PHP endpoint, SQL parameter, request, or implementation.

## Remediation

The following defensive recommendations are retained from the source and are not evidence that remediation has been applied.

- Avoid exposing sensitive files or directories through the web server.
- Validate and restrict file access functionality.
- Do not store passwords in accessible local files.
- Use parameterized queries to prevent SQL injection.
- Restrict database permissions based on least privilege.
- Monitor abnormal authentication and database access attempts.

## Lessons Learned

### Key takeaways

- Failed credential attacks do not always mean the discovered username is useless.
- Directory enumeration can reveal both usernames and application files.
- Local file access vulnerabilities can expose sensitive files and credentials.
- PHP file handling must be carefully validated to prevent unintended file access.
- Burp Suite is useful for capturing accurate requests before testing with SQLmap.
- SQL injection can expose sensitive database content, including user credentials.
- Multiple small findings can be chained into a complete compromise path.

### Reflection

This lab helped me understand how enumeration findings can be reused across different stages of an assessment.

Even though the initial credential testing attempt did not work, the discovered username and directory information still helped guide further testing. The most important lesson was that failed paths can still provide useful context.

I also practiced using Burp Suite and SQLmap together, which helped me understand the value of capturing real application requests before automated SQL injection testing.

### Skills practiced

- Port scanning
- Web application enumeration
- Directory discovery
- Username discovery from exposed content
- Basic credential testing
- Local file access testing
- Analysing PHP file access behaviour
- Capturing HTTP requests with Burp Suite
- Testing SQL injection with SQLmap
- Extracting credentials from a vulnerable database in a lab environment

## Evidence and Limitations

- Source: [Original learning note](https://github.com/Dongeun-Kang/cybersecurity/blob/3d27c869c37f6b8550ead2fa332fdb04b18dde9c/labs/THM_Recruit.md) (immutable pre-migration revision).
- Documentation migration: 2026-09-16; this is not the testing date.
- Deferred: local-file endpoint/path and SQL injection request, parameter, and response evidence are missing.
- The source does not contain raw session transcripts or screenshots. Omitted flags, credentials, private keys, and other sensitive values remain omitted.

## Original Disclaimer

This note is for educational purposes only.  
All activity was performed in a legal, controlled TryHackMe lab environment.

Flags, credentials, passwords, and full exploit commands are intentionally omitted.
