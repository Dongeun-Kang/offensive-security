# Blue

This is a lab for practicing SMB enumeration and exploitation against a vulnerable Windows machine.

The objective of this lab is to enumerate the target, identify an SMB vulnerability, exploit the vulnerability using Metasploit, dump user password hashes, and recover credentials.

Firstly, I performed basic enumeration against the target using RustScan.

The scan showed three open ports with port numbers under 1000: `135`, `139`, and `445`.

Port `445` was open and running Microsoft SMB on `Windows Server 2012 R2 Datacenter 9600`.

<img width="778" height="67" alt="image" src="https://github.com/user-attachments/assets/0819f4f4-be16-4833-bcee-0c6287e96534" />

Since SMB was available on the target, I checked for known vulnerabilities affecting this version of Windows and identified `MS17-010 (EternalBlue)`.

I started Metasploit using `msfconsole` and searched for an exploit for MS17-010.

The exploit module was:

`exploit/windows/smb/ms17_010_eternalblue`

After selecting the module, I checked the available options and configured the required `RHOSTS` value with the target IP address.

I then executed the exploit against the target.

The exploit was successful and provided an elevated Meterpreter session on the Windows machine.

Within the Meterpreter session, I used `hashdump` to dump the local account password hashes.

<img width="702" height="72" alt="image" src="https://github.com/user-attachments/assets/12e60f44-8d9e-4337-8343-c2354bf4a0a3" />

From the dumped hashes, I identified a non-default user named `Jon`.

I copied Jon's NTLM password hash into a file and used a password cracking tool to recover the plaintext password.

<img width="785" height="170" alt="image" src="https://github.com/user-attachments/assets/5470490f-0f83-4d65-92cf-d2ca3a3407c8" />

The password was successfully cracked as `alqfna22`.

Using the access obtained through EternalBlue and the recovered credentials, I enumerated the compromised system and located the required flags.

After collecting the flags, I successfully completed the lab.
