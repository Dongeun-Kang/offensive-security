# ToolsRus

## Summary

- Platform: TryHackMe
- Category: Web Security / Enumeration / Basic Authentication / Tomcat Manager / Metasploit
- Status: Completed, as recorded in the source note
- Difficulty: Not recorded
- Report: Not created; see evidence limitations below.

This lab focused on web service enumeration, discovering hidden directories, identifying authentication-protected areas, and using discovered information to gain access to a misconfigured Tomcat Manager interface in a controlled lab environment.

The main objective was to practice a structured enumeration workflow and understand how weak credentials and exposed management interfaces can lead to system compromise.

## Environment

- Target: Authorized TryHackMe lab; target IP and testing date were not recorded.
- Wordlists or supporting files: Only those explicitly named in the workflow below; other values were not recorded.

### Tools

- Nmap
- Gobuster
- Hydra
- Metasploit
- Web browser (Firefox)

## Enumeration

Web directory discovery exposed a username. Basic Authentication testing yielded access to a page pointing to a relocated service and Tomcat Manager.

## Exploitation

### Recorded workflow

The following workflow is retained from the source note, including enumeration context and unsuccessful attempts. Commands and outcomes were documented by the author; they were not rerun during migration. Placeholder values are not recovered secrets.

1. Performed an Nmap scan and identified an open web service.
2. Visited the website to understand the exposed application.
3. Used directory enumeration to discover hidden paths.
4. Found a username from one of the discovered directories.
5. Tested Basic Authentication using the discovered username in the lab environment.
6. Retrieved valid credentials and accessed a restricted page.
7. Identified that the application had moved to another port.
8. Accessed the relocated service and found a Tomcat Manager interface.
9. Used Metasploit with the valid manager credentials to exploit the vulnerable Tomcat setup.
10. Gained shell access and retrieved the lab flag.

## Root Cause

The narrative describes valid manager credentials enabling code execution through Tomcat. Weak credential protection and reachable management functionality are implicated; a particular Tomcat software vulnerability is not established.

## Remediation

The following defensive recommendations are retained from the source and are not evidence that remediation has been applied.

- Do not expose administrative interfaces publicly.
- Avoid weak or default credentials.
- Restrict access to management portals.
- Monitor failed authentication attempts.
- Remove unnecessary web directories and exposed information.
- Apply least privilege to service accounts.

## Lessons Learned

### Key takeaways

- Initial port scanning is essential for identifying exposed services.
- Directory enumeration can reveal useful information such as usernames, hidden pages, or service hints.
- Basic Authentication can be weak if poor credentials are used.
- Exposed management interfaces, such as Tomcat Manager, are high-risk when accessible from the network.
- Valid credentials can often be more dangerous than a software vulnerability by itself.
- Metasploit is useful for controlled lab exploitation, but the vulnerability and configuration should still be understood manually.

### Reflection

This lab helped me understand how enumeration results can connect together across multiple stages.

A discovered directory led to a username, the username helped with authentication testing, the authenticated page revealed another service location, and the relocated service exposed a vulnerable Tomcat Manager interface.

The main lesson was that successful exploitation often depends on chaining small findings rather than relying on a single obvious vulnerability.

### Skills practiced

- Port scanning
- Web application enumeration
- Directory brute-forcing
- Identifying usernames from exposed content
- Testing Basic Authentication
- Using Hydra for credential testing in a lab environment
- Enumerating relocated web services
- Using Metasploit against a vulnerable Tomcat Manager setup
- Basic post-exploitation navigation

## Evidence and Limitations

- Source: [Original learning note](https://github.com/Dongeun-Kang/cybersecurity/blob/3d27c869c37f6b8550ead2fa332fdb04b18dde9c/labs/THM_ToolsRus.md) (immutable pre-migration revision).
- Documentation migration: 2026-09-16; this is not the testing date.
- Deferred: service endpoint/port, authentication evidence, Metasploit module, and execution output are missing.
- The source does not contain raw session transcripts or screenshots. Omitted flags, credentials, private keys, and other sensitive values remain omitted.

## Original Disclaimer

This note is for educational purposes only.  
All activity was performed in a legal, controlled TryHackMe lab environment.

Flags, credentials, passwords, and full exploit commands are intentionally omitted.
