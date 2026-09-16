# Kenobi

## Summary

- Platform: TryHackMe
- Category: Network Enumeration / SMB Enumeration / NFS Enumeration / FTP Exploitation / Public Exploit Research / Linux Privilege Escalation / PATH Hijacking
- Status: Completed, as recorded in the source note
- Difficulty: Not recorded
- Report: [Assessment report](../../../reports/tryhackme/kenobi-pentest-report.md)

This lab focused on enumerating multiple exposed network services, identifying accessible SMB shares, discovering useful information from an anonymous share, researching a vulnerable FTP service, abusing ProFTPD’s `mod_copy` functionality, gaining SSH access with a discovered private key, and escalating privileges through PATH hijacking.

The main objective was to scan the target, identify exposed services, enumerate SMB and NFS, investigate FTP service information, research a known ProFTPD vulnerability, copy a sensitive file into a mountable location, obtain SSH access, and escalate privileges to root by abusing an insecure SUID binary.

## Environment

- Target: Authorized TryHackMe lab; target IP and testing date were not recorded.
- Wordlists or supporting files: Only those explicitly named in the workflow below; other values were not recorded.

### Tools

* RustScan
* Nmap
* SMBClient
* Netcat
* SearchSploit
* NFS mount
* SSH
* find
* strings
* chmod
* export
* Terminal

## Enumeration

The recorded ports were 21, 22, 80, 111, 139, 445, 2049, 33835, 36171, 43053, and 46323. SMB exposed an anonymous share; NFS exported `/var`; FTP was identified as ProFTPD 1.3.5.

## Exploitation

### Recorded workflow

The following workflow is retained from the source note, including enumeration context and unsuccessful attempts. Commands and outcomes were documented by the author; they were not rerun during migration. Placeholder values are not recovered secrets.

1. Performed initial full port discovery with RustScan.

   ```bash
   rustscan -a $target -- -sC -sV -oN nmap_details.txt
   ```

   RustScan was used to quickly identify open ports. The `--` passed additional arguments to Nmap, allowing service/version detection and default script scanning to run after RustScan discovered the ports.

2. Identified multiple open ports on the target.

   The scan found the following ports open:

   ```text
   21, 22, 80, 111, 139, 445, 2049, 33835, 36171, 43053, 46323
   ```

   This showed that the machine exposed several services, including FTP, SSH, HTTP, RPC, SMB, and NFS-related ports.

3. Focused on SMB enumeration because ports `139` and `445` were open.

   ```bash
   nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse $target
   ```

   This command attempted to enumerate SMB shares and users using Nmap NSE scripts.

   The script scan did not work properly, likely because Kenobi is an older lab and the SMB service/script compatibility caused issues. Instead of stopping there, SMB enumeration was continued manually with `smbclient`.

4. Listed available SMB shares anonymously.

   ```bash
   smbclient -L //$target/ -N
   ```

   The `-L` option listed available shares on the target.
   The `-N` option attempted authentication without a password.

   Three shares were discovered, including:

   ```text
   print$
   anonymous
   IPC$
   ```

   The `anonymous` share was especially interesting because it suggested unauthenticated access may be allowed.

5. Connected to the anonymous SMB share.

   ```bash
   smbclient //$target/anonymous
   ```

   After connecting to the share, the available files were inspected.

6. Found a log file inside the anonymous share.

   The share contained a log file. The file was downloaded and reviewed locally.

   The log revealed useful service information, including evidence that anonymous access was also possible on the FTP service.

   This was important because SMB enumeration did not directly give a shell, but it gave information that helped guide the next enumeration step.

7. Enumerated NFS because port `111` and `2049` were open.

   ```bash
   nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount $target
   ```

   This command used Nmap NFS scripts to list exported directories, file system information, and available mount points.

   NFS enumeration showed that a shared directory was available. This became important later because the FTP vulnerability could copy files into a location accessible through NFS.

8. Performed FTP banner grabbing with Netcat.

   ```bash
   nc $target 21
   ```

   Netcat was used to connect directly to FTP on port `21`.

   The banner displayed the FTP service and version. Although this version had already appeared in the detailed Nmap scan, banner grabbing confirmed the service manually.

9. Researched the FTP service version using SearchSploit.

   ```bash
   searchsploit proftpd 1.3.5
   ```

   SearchSploit was used to check whether the identified ProFTPD version had known public vulnerabilities.

   A relevant vulnerability was found involving the `mod_copy` module.

10. Identified the purpose of the vulnerable `mod_copy` module.

The `mod_copy` module implements FTP commands such as:

```text
SITE CPFR <path>
SITE CPTO <path>
```

`SITE CPFR` selects the source file to copy.
`SITE CPTO` defines the destination path.

In this lab, the vulnerability allowed files to be copied from one location to another on the server through the FTP service.

11. Connected to FTP again to exploit the vulnerable copy functionality.

```bash
nc $target 21
```

After connecting to the FTP service, the vulnerable `SITE CPFR` and `SITE CPTO` commands were used.

12. Copied the target user’s SSH private key into an accessible location.

```text
SITE CPFR /home/kenobi/.ssh/id_rsa
SITE CPTO /var/tmp/id_rsa
```

The private key was copied from the user’s SSH directory into `/var/tmp`.

This worked because the vulnerable FTP service allowed server-side file copying, and `/var` was accessible through NFS.

13. Created a local mount directory for the NFS share.

```bash
mkdir kenobiNFS
```

This directory was created locally to serve as the mount point.

14. Mounted the exported NFS directory.

```bash
mount $target:/var kenobiNFS
```

The target’s `/var` directory was mounted locally.

Since the SSH private key had been copied to `/var/tmp/id_rsa`, it could now be accessed from the mounted NFS directory.

15. Retrieved the SSH private key from the mounted NFS share.

The copied private key was found in the mounted `/var/tmp/` path.

The key content itself is intentionally omitted from this note.

16. Used the private key to log in through SSH.

SSH access was obtained using the discovered private key.

The exact private key content and flag values are intentionally omitted.

17. Retrieved the user flag.

After logging in as the target user, the user flag was found.

18. Attempted to check sudo privileges.

```bash
sudo -l
```

This required the user’s password, which was not known.

Because `sudo -l` could not be used, another privilege escalation path was needed.

19. Searched for SUID binaries.

```bash
find / -perm -u=s -type f 2>/dev/null
```

This command searched the filesystem for files with the SUID bit set.

`-perm -u=s` searches for files where the user SUID permission is enabled.
`-type f` limits the result to files.
`2>/dev/null` hides permission-denied errors.

20. Found a suspicious SUID binary.

A suspicious binary was found:

```text
/usr/bin/menu
```

This binary was interesting because custom or unusual SUID binaries are common privilege escalation targets in Linux labs.

21. Executed the suspicious binary to observe its behavior.

```bash
/usr/bin/menu
```

Running the binary showed that it provided a menu and executed system commands in the background.

22. Inspected the binary with `strings`.

```bash
strings /usr/bin/menu
```

The `strings` command extracts readable text from a binary.

The output showed that the binary called `curl` without using the full absolute path.

This was important because calling `curl` instead of `/usr/bin/curl` meant the system would search for `curl` using the current `PATH` environment variable.

23. Identified PATH hijacking as the privilege escalation method.

PATH hijacking is possible when a privileged program runs a command without specifying its full path.

If an attacker can place a malicious file with the same command name earlier in the `PATH`, the privileged program may execute the attacker-controlled file instead of the legitimate binary.

24. Created a fake `curl` command in `/tmp`.

```bash
echo /bin/sh > /tmp/curl
```

This created a file named `curl` that would execute `/bin/sh`.

25. Made the fake `curl` executable.

```bash
chmod 777 /tmp/curl
```

This gave the file read, write, and execute permissions.

In a real environment, using overly permissive permissions such as `777` is insecure, but in this lab it was used to make sure the file could be executed.

26. Modified the `PATH` variable.

```bash
export PATH=/tmp:$PATH
```

This placed `/tmp` at the beginning of the `PATH`.

As a result, when the vulnerable SUID binary called `curl`, the system would find `/tmp/curl` first instead of the legitimate `curl` binary.

27. Executed the vulnerable SUID binary again.

```bash
/usr/bin/menu
```

When the binary attempted to run `curl`, it executed the attacker-controlled `/tmp/curl` file.

28. Gained a root shell.

Because `/usr/bin/menu` had elevated privileges, the shell spawned through the hijacked `curl` command inherited root privileges.

29. Retrieved the root flag.

After gaining root access, the root flag was found.

## Root Cause

The recorded chain combined FTP server-side copying of a private key into an NFS-readable location with a SUID program that resolved `curl` through an attacker-influenced `PATH`.

## Remediation

The following defensive recommendations are retained from the source and are not evidence that remediation has been applied.

* Anonymous SMB access should be disabled unless there is a clear business requirement.
* Shared directories should not expose logs or configuration files that reveal service details.
* FTP services should not allow anonymous access unless strictly necessary.
* Outdated services such as vulnerable ProFTPD versions should be patched or removed.
* Dangerous modules such as ProFTPD `mod_copy` should be disabled when not required.
* Sensitive files such as SSH private keys should never be accessible to service accounts.
* NFS exports should be restricted to trusted clients only.
* NFS permissions should be carefully configured to prevent sensitive file exposure.
* Services should not allow one exposed protocol to move sensitive files into another accessible location.
* SUID binaries should be reviewed regularly.
* Custom SUID binaries are high-risk and should be avoided unless absolutely necessary.
* Privileged programs should always call system binaries using absolute paths.
* Environment variables such as `PATH` should not be trusted in privileged execution contexts.
* The principle of least privilege should be applied across SMB, FTP, NFS, SSH, and local binaries.
* Regular privilege escalation audits can help identify insecure SUID binaries before attackers abuse them.

## Lessons Learned

### Key takeaways

* RustScan is useful for quickly discovering open ports, while Nmap options can be passed through for detailed service enumeration.
* When Nmap scripts fail, manual enumeration should continue instead of relying on one tool.
* SMB anonymous shares can expose files that reveal useful service information.
* Information disclosure from one service can guide exploitation of another service.
* NFS exports can become dangerous when sensitive files are copied into mountable directories.
* FTP banner grabbing is useful for manually confirming service versions.
* SearchSploit helps quickly identify known public vulnerabilities for specific service versions.
* ProFTPD `mod_copy` can be dangerous because it allows server-side file copying.
* A vulnerability does not always directly give a shell; sometimes it gives access to a sensitive file that enables the next step.
* SSH private keys are highly sensitive and should never be readable or movable by exposed services.
* If `sudo -l` requires a password, SUID enumeration is a strong alternative privilege escalation path.
* Custom SUID binaries should always be investigated carefully.
* `strings` is useful for identifying commands executed inside binaries.
* Using commands without absolute paths inside privileged programs can lead to PATH hijacking.
* PATH hijacking can escalate privileges when a SUID binary executes attacker-controlled commands.

### Reflection

This lab helped me understand how enumeration across multiple services can be chained together to gain access and escalate privileges.

At first, I used RustScan with Nmap options to identify open ports and service details. The target exposed several services, including FTP, SSH, SMB, RPC, and NFS. Since SMB ports were open, I attempted to enumerate SMB shares with Nmap scripts, but the script scan did not work properly. Instead of depending only on Nmap, I used `smbclient` manually and found accessible shares, including an anonymous share.

The anonymous SMB share contained a log file. This file did not directly give access, but it provided useful information about the FTP service and showed that anonymous FTP access was possible. This reinforced that enumeration is not only about finding credentials or flags immediately. Sometimes the value of enumeration is finding information that supports the next attack step.

After that, I enumerated NFS and confirmed that a mountable directory was available. I then checked the FTP service manually with Netcat and confirmed the ProFTPD version. Using SearchSploit, I found a vulnerability related to ProFTPD 1.3.5 and the `mod_copy` module. The important part was understanding what the module did: it allowed files to be copied from one server-side path to another using `SITE CPFR` and `SITE CPTO`.

I used this behavior to copy the user’s SSH private key into a location that was accessible through the NFS-mounted directory. After mounting the NFS share locally, I was able to retrieve the copied private key and use it to log in through SSH. This showed how SMB, FTP, and NFS information could be combined into one exploitation chain.

After gaining user access, I tried `sudo -l`, but it required the user’s password. Since I did not know the password, I moved to SUID enumeration with `find / -perm -u=s -type f 2>/dev/null`. This revealed a suspicious binary, `/usr/bin/menu`. Running the binary showed menu-like behavior, and using `strings` revealed that it executed `curl` without an absolute path.

Because the binary called `curl` instead of `/usr/bin/curl`, it was vulnerable to PATH hijacking. I created a fake `curl` file in `/tmp`, made it executable, and modified the `PATH` so `/tmp` appeared first. When I executed `/usr/bin/menu` again, the binary ran my fake `curl` instead of the real one, resulting in a root shell.

This lab reinforced that privilege escalation often depends on small implementation mistakes. A single missing absolute path inside a privileged binary can be enough to compromise the whole system.

### Skills practiced

* Full port scanning
* Service enumeration
* SMB share enumeration
* Anonymous SMB access
* NFS enumeration
* NFS mounting
* FTP banner grabbing
* Vulnerability research with SearchSploit
* Understanding ProFTPD `mod_copy`
* Exploiting file copy functionality in a lab environment
* SSH login with a private key
* Linux post-exploitation enumeration
* SUID binary discovery
* Binary analysis with `strings`
* PATH hijacking
* Linux privilege escalation

## Evidence and Limitations

- Source: [Original learning note](https://github.com/Dongeun-Kang/cybersecurity/blob/3d27c869c37f6b8550ead2fa332fdb04b18dde9c/labs/THM_Kenobi.md) (immutable pre-migration revision).
- Documentation migration: 2026-09-16; this is not the testing date.
- Report: exact file-copy commands, NFS mount path, SUID binary, PATH manipulation, and reported root-shell outcome are documented.
- The source does not contain raw session transcripts or screenshots. Omitted flags, credentials, private keys, and other sensitive values remain omitted.

## Original Disclaimer

This note is for educational purposes only.

All activity was performed in a legal, controlled TryHackMe lab environment.

Flags, credentials, private key contents, hashes, and sensitive details are intentionally omitted.
