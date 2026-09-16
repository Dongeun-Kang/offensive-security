# Epoch

## Summary

- Platform: TryHackMe
- Category: Web Security / Command Injection / Linux Enumeration
- Status: Completed, as recorded in the source note
- Difficulty: Not recorded
- Report: Not created; see evidence limitations below.

This lab focused on identifying and exploiting a basic command injection vulnerability in a web application.

The application accepted user input and passed it to the underlying operating system without proper validation, allowing additional shell commands to be executed.

## Environment

- Target: Authorized TryHackMe lab; target IP and testing date were not recorded.
- Wordlists or supporting files: Only those explicitly named in the workflow below; other values were not recorded.

### Tools

- Web browser (Firefox)
- Python HTTP server
- Netcat
- curl

## Enumeration

The application accepted input that was tested first with a normal value and then with a command separator.

## Exploitation

### Recorded workflow

The following workflow is retained from the source note, including enumeration context and unsuccessful attempts. Commands and outcomes were documented by the author; they were not rerun during migration. Placeholder values are not recovered secrets.

1. Tested normal input to understand the application response.
2. Used a command separator to test for command injection.
3. Confirmed command execution with a basic identity check.
4. Established a reverse shell for interactive access.
5. Performed basic Linux enumeration.
6. Checked environment variables and found lab-specific information.

### Tools Used

- Web browser (Firefox)
- Python HTTP server
- Netcat
- curl

## Root Cause

User input reached shell execution without a safe separation between data and commands, according to the source narrative. Backend implementation and the vulnerable parameter were not recorded.

## Remediation

The following defensive recommendations are retained from the source and are not evidence that remediation has been applied.

- Never pass user input directly into system commands.
- Validate and sanitize all user-controlled input.
- Avoid using shell execution functions unless absolutely necessary.
- Apply allowlists for expected input values instead of relying only on blocklists.
- Run web applications with least-privilege service accounts.
- Do not store sensitive information in environment variables.
- Monitor for suspicious command execution patterns.

## Lessons Learned

### Key takeaways

- Command injection occurs when user input is passed to system commands without proper validation.
- Simple payloads are useful for safely confirming command execution.
- A reverse shell provides interactive access after successful command execution.
- Environment variables can contain useful information during post-exploitation enumeration.
- Sensitive data should not be stored in environment variables.

### Reflection

This lab helped me understand how command injection can lead to remote command execution through unsafe input handling.

I also learned that environment variables should be checked during enumeration, as they may expose sensitive or lab-specific information.

At first, I used a hosted reverse shell script, but I later realised that a direct Bash reverse shell could also be used. This helped me better understand what the payload was doing instead of relying only on a prepared script.

### Skills practiced

- Testing user input for command injection
- Observing application behaviour with simple payloads
- Confirming remote command execution
- Establishing a reverse shell in a lab environment
- Inspecting environment variables on Linux

## Evidence and Limitations

- Source: [Original learning note](https://github.com/Dongeun-Kang/cybersecurity/blob/3d27c869c37f6b8550ead2fa332fdb04b18dde9c/labs/THM_Epoch.md) (immutable pre-migration revision).
- Documentation migration: 2026-09-16; this is not the testing date.
- Deferred: input endpoint/parameter, submitted payload, and returned command output are missing.
- The source does not contain raw session transcripts or screenshots. Omitted flags, credentials, private keys, and other sensitive values remain omitted.

## Original Disclaimer

This note is for educational purposes only.  
All activity was performed in a legal, controlled lab environment.

Flags, credentials, and full exploit steps are intentionally omitted.
