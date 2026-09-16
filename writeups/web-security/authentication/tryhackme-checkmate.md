# Checkmate

## Summary

- Platform: TryHackMe
- Category: Password Security / Web Authentication / Brute Force / Custom Wordlist Generation / Hash Cracking / SSH Password Attack
- Status: Completed, as recorded in the source note
- Difficulty: Not recorded
- Report: [Assessment report](../../../reports/tryhackme/checkmate-pentest-report.md)

This lab focused on identifying weak password patterns across multiple authentication levels and using different password attack techniques to demonstrate how predictable passwords can be compromised.

The main objective was to assess Marco’s password usage across several services, identify how each password was likely generated, build suitable wordlists, and perform controlled password attacks against web login forms, hashes, and SSH authentication.

The lab demonstrated that weak passwords are not only about using common passwords. They can also be created from company keywords, personal information, predictable patterns, years, symbols, and reused naming conventions.

## Environment

- Target: Authorized TryHackMe lab; target IP and testing date were not recorded.
- Wordlists or supporting files: Only those explicitly named in the workflow below; other values were not recorded.

### Tools

* Hydra
* CeWL
* CUPP
* Hashcat
* Crunch
* RockYou wordlist
* Web browser
* View Source
* Terminal
* SSH

## Enumeration

The source records three HTTP login forms on ports 5001–5003, a hash-like public image filename, and an SSH service with a predictable password pattern.

## Exploitation

### Recorded workflow

The following workflow is retained from the source note, including enumeration context and unsuccessful attempts. Commands and outcomes were documented by the author; they were not rerun during migration. Placeholder values are not recovered secrets.

1. Added the target domains to local host resolution.

   ```bash
   echo "$target <domain1> <domain2> <domain3>" | sudo tee -a /etc/hosts
   ```

   The lab used multiple domains mapped to the same target IP address.

   Adding these domains to `/etc/hosts` allowed the browser and tools to resolve each lab domain correctly.

---

### Level 1 — Default or Common Credential Weakness

2. Accessed the first web application.

   The first login page suggested that a default or common credential might be in use.

   Since the username appeared to be known or guessable, the attack focused on testing common passwords against a single username.

3. Used Hydra against the HTTP POST login form.

   ```bash
   hydra -l admin -P /usr/share/wordlists/rockyou.txt $domain1 http-post-form "/login:username=^USER^&password=^PASS^:Invalid Credential." -s 5001 -t 4 -V
   ```

   Command breakdown:

   ```bash
   -l admin
   ```

   Specifies a single username: `admin`.

   ```bash
   -P /usr/share/wordlists/rockyou.txt
   ```

   Specifies the password wordlist. `rockyou.txt` is commonly used for password-cracking practice because it contains many real-world leaked passwords.

   ```bash
   $domain1
   ```

   Specifies the target domain.

   ```bash
   http-post-form
   ```

   Tells Hydra that the target is an HTTP POST login form.

   ```bash
   "/login:username=^USER^&password=^PASS^:Invalid Credential."
   ```

   Defines the login request structure.

   `^USER^` is replaced by Hydra with the username.
   `^PASS^` is replaced by Hydra with each password candidate.
   `Invalid Credential.` tells Hydra what failed login response looks like.

   ```bash
   -s 5001
   ```

   Specifies the service port.

   ```bash
   -t 4
   ```

   Runs four parallel tasks.

   ```bash
   -V
   ```

   Enables verbose output so each attempt can be observed.

4. Successfully cracked the login password.

   The actual password is intentionally omitted.

---

### Level 2 — Company Keyword-Based Password

5. Accessed the second web application.

   The second level suggested that the password was related to company-specific keywords.

   Instead of using only a generic wordlist, a custom wordlist was created from the target website content.

6. Generated a company-specific wordlist with CeWL.

   ```bash
   cewl -d 2 -m 6 --lowercase -w keywords.txt http://$domain2:5002
   ```

   Command breakdown:

   ```bash
   -d 2
   ```

   Crawls the website up to a depth of 2 links.

   ```bash
   -m 6
   ```

   Only collects words with a minimum length of 6 characters.

   ```bash
   --lowercase
   ```

   Converts collected words to lowercase.

   ```bash
   -w keywords.txt
   ```

   Saves the generated wordlist into `keywords.txt`.

   ```bash
   http://$domain2:5002
   ```

   Specifies the target website to crawl.

7. Used Hydra with the generated keyword wordlist.

   ```bash
   hydra -l marco -P keywords.txt $domain2 http-post-form "/login:username=^USER^&password=^PASS^:Invalid Credential." -s 5002 -t 4 -V
   ```

   This attack tested whether Marco’s password was based on words found on the company website.

8. Successfully cracked the login password.

   The actual password is intentionally omitted.

---

### Level 3 — Personal Information-Based Password

9. Accessed the third web application.

   The password appeared to be generated from personal information.

   This is a common real-world weakness because users often build passwords from names, birthdays, nicknames, pets, companies, job roles, or other personal details.

10. Gathered personal information from the previous domain.

Relevant personal details were collected from the available web pages.

The actual details are intentionally omitted.

11. Generated a custom wordlist with CUPP.

```bash
python3 cupp.py -i
```

CUPP is a tool that creates custom password wordlists based on personal information.

The `-i` option starts interactive mode, where information such as names, nicknames, dates, partner names, pet names, and keywords can be entered.

12. Used the generated CUPP wordlist with Hydra.

```bash
hydra -l marco -P marco.txt $domain3 http-post-form "/login:username=^USER^&password=^PASS^:Invalid Credential." -s 5003 -t 4 -V
```

This attack tested whether the password was based on Marco’s personal information.

13. Successfully cracked the login password.

The actual password is intentionally omitted.

---

### Level 4 — Hash-Based Password Weakness

14. Investigated the fourth level.

The user icon image was stored using a filename that appeared to be a SHA-256 hash.

The image file followed a pattern similar to:

```text
<sha256_hash>.png
```

15. Inspected the page source.

The image filename was found by viewing the page source.

This revealed the hash value through the image path.

16. Saved the hash into a file.

```bash
echo "<hash>" > hash.txt
```

The actual hash is intentionally omitted.

17. Cracked the hash with Hashcat.

```bash
hashcat -m 1400 -a 0 hash.txt rockyou.txt
```

Command breakdown:

```bash
-m 1400
```

Specifies the hash mode. Mode `1400` is used for raw SHA-256.

```bash
-a 0
```

Specifies a straight dictionary attack.

```bash
hash.txt
```

Contains the target hash.

```bash
rockyou.txt
```

Contains the password candidates.

18. Successfully cracked the hash.

The cracked plaintext is intentionally omitted.

---

### Level 5 — Pattern-Based SSH Password

19. Investigated the final level.

The SSH password appeared to follow a predictable structure:

```text
company keyword + year/number pattern + special character
```

The observed pattern was similar to:

```text
Security20??!
```

This meant a full brute-force attack was unnecessary. A targeted pattern-based wordlist could be generated instead.

20. Created a custom pattern wordlist with Crunch.

```bash
crunch 13 13 -t Security20%%! > passwords.txt
```

Command breakdown:

```bash
13 13
```

Sets the minimum and maximum password length to 13 characters.

```bash
-t Security20%%!
```

Defines the password pattern.

In Crunch, `%` represents a digit from `0` to `9`.

Therefore, this pattern generates passwords such as:

```text
Security2000!
Security2001!
Security2002!
...
```

The actual matched password is intentionally omitted.

21. Repeated pattern generation as needed.

Additional rules or patterns were generated based on the observed password structure.

This avoided wasting time on unrelated password candidates.

22. Used Hydra against SSH.

```bash
hydra -l marco -P passwords.txt ssh://$target
```

Command breakdown:

```bash
-l marco
```

Specifies the SSH username.

```bash
-P passwords.txt
```

Specifies the generated password list.

```bash
ssh://$target
```

Tells Hydra to attack the SSH service on the target.

23. Successfully cracked the SSH password.

The actual password is intentionally omitted.

## Root Cause

The recorded attacks relied on common, company-derived, personal-information-derived, and patterned passwords, plus a publicly exposed SHA-256 value that was recovered offline. Rate-limit absence and the backend password-storage scheme were not independently established.

## Remediation

The following defensive recommendations are retained from the source and are not evidence that remediation has been applied.

* Default credentials should always be changed before deployment.
* Passwords should not be based on company names, public keywords, personal details, or predictable patterns.
* Public website content can be used to create targeted password wordlists.
* Personal information exposure increases the risk of password guessing attacks.
* Applications should limit failed login attempts to reduce brute-force risk.
* Account lockout or rate limiting should be implemented for web login forms and SSH.
* Multi-factor authentication should be used for sensitive accounts.
* Passwords should be long, unique, and randomly generated.
* SSH password authentication should be disabled where possible in favor of key-based authentication.
* Exposed hashes should be treated as sensitive information.
* Sensitive values should not be used as public filenames.
* Developers should avoid exposing implementation details through page source, static file names, or predictable naming conventions.
* Security teams should test password policies against real-world user behavior, not only against formal complexity rules.
* A password that satisfies complexity requirements can still be weak if it follows a predictable pattern.

## Lessons Learned

### Key takeaways

* Password attacks should be based on observed evidence, not random guessing.
* Default or common credentials are still a major security weakness.
* `rockyou.txt` is useful for common-password testing, but it should not be the only approach.
* CeWL is useful when passwords are likely based on company-specific words.
* CUPP is useful when passwords are likely based on personal information.
* Page source review can reveal hidden or useful implementation details.
* File names can accidentally expose hashes or sensitive information.
* Hashcat mode selection matters. For raw SHA-256, mode `1400` is used.
* Crunch is useful when the password follows a known structure.
* Pattern-based attacks are more efficient than full brute-force attacks.
* Weak password construction often follows predictable human behavior.
* Password reuse and predictable password formats make compromise easier.
* Each level required a different attack strategy based on the clue and context.

### Reflection

This lab helped me understand that password attacks are not only about running a large wordlist against a login page. A better approach is to first observe the application, identify clues, understand the likely password pattern, and then choose the most efficient attack method.

In Level 1, the weakness was a default or common credential. Since the username was known, a simple Hydra attack using `rockyou.txt` was enough. This showed why common passwords are dangerous, especially when used with predictable usernames like `admin`.

In Level 2, the password was based on company-related keywords. Instead of using only a generic wordlist, I used CeWL to crawl the target website and generate a custom wordlist from the company’s own content. This showed how attackers can use public-facing website content to create targeted password lists.

In Level 3, the password was based on personal information. I collected relevant information from the available web content and used CUPP to generate a personalized wordlist. This showed how personal details can become dangerous when users include them in passwords.

In Level 4, I found that an icon image filename was stored as a SHA-256 hash. By viewing the page source, I extracted the hash and cracked it with Hashcat using mode `1400`. This showed that even filenames can leak useful information if developers store sensitive values in predictable or exposed ways.

In Level 5, the SSH password followed a predictable pattern using a company keyword, a year-like number, and an exclamation mark. Instead of brute-forcing every possible password, I used Crunch to generate a focused wordlist based on the observed pattern. This made the attack much more efficient.

Overall, this lab taught me that strong password assessment requires pattern recognition. The most important step is not immediately running tools, but understanding how the password was likely created.

### Skills practiced

* Editing local host resolution
* Web application enumeration
* Login form analysis
* Brute-force attack with Hydra
* Dictionary attack
* Company-specific wordlist generation
* CeWL-based crawling
* Personal information gathering
* Custom wordlist generation with CUPP
* Source code inspection
* Hash extraction from filenames
* SHA-256 hash cracking with Hashcat
* Pattern-based password generation with Crunch
* SSH password attack
* Password pattern analysis
* Controlled exploitation documentation

## Evidence and Limitations

- Source: [Original learning note](https://github.com/Dongeun-Kang/cybersecurity/blob/3d27c869c37f6b8550ead2fa332fdb04b18dde9c/labs/THM_checkmate.md) (immutable pre-migration revision).
- Documentation migration: 2026-09-16; this is not the testing date.
- Report: form structure, tools, hash mode, wordlist construction, and successful password-recovery outcomes are recorded.
- The source does not contain raw session transcripts or screenshots. Omitted flags, credentials, private keys, and other sensitive values remain omitted.

## Original Disclaimer

This note is for educational purposes only.

All activity was performed in a legal, controlled TryHackMe lab environment.

Passwords, hashes, flags, personal details, and sensitive values are intentionally omitted.
