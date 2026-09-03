# Vulnversity

This is a lab for practicing network enumeration, web directory enumeration, file upload exploitation, reverse shells, and Linux privilege escalation.

The objective of this lab is to enumerate the target, identify a vulnerable file upload functionality, gain an initial shell by uploading a reverse shell payload, and escalate privileges to root.

## Enumeration

Firstly, I performed basic enumeration against the target using `RustScan` and `Nmap`.

The scan revealed **6 open ports**, providing several potential services to investigate.

I then focused on the web server and used `Gobuster` to enumerate hidden directories.

During directory enumeration, I discovered a directory containing a file upload functionality.

This immediately presented a potential attack vector because, if the server accepted executable files, it might be possible to upload and execute a reverse shell payload.

<img width="520" height="247" alt="image" src="https://github.com/user-attachments/assets/8234857f-b2ed-4cc8-bf53-13c513c6c1b4" />

## File Upload Enumeration

I used `Burp Suite` to intercept the file upload request and test which file extensions were accepted by the application.

After testing multiple PHP-related extensions, I discovered that the server accepted the `.phtml` extension.

Since `.phtml` files can be interpreted as PHP by the web server, I used `msfvenom` to generate a `.phtml` reverse shell payload.

<img width="442" height="55" alt="image" src="https://github.com/user-attachments/assets/e797b61a-25bc-4218-9905-f79f9481d6e3" />

## Initial Access

Before executing the uploaded payload, I started a Netcat listener on port `1337` to receive the reverse connection.

```bash
nc -lvnp 1337
```

<img width="427" height="47" alt="image" src="https://github.com/user-attachments/assets/3efd26ab-d5ee-4802-9670-395a7f079dd9" />

I then uploaded the `.phtml` payload and accessed the uploaded file through the browser.

The server executed the payload and successfully connected back to my Netcat listener, giving me a shell on the target system.

After gaining initial access, I enumerated the filesystem and located the **user flag**.

<img width="265" height="67" alt="image" src="https://github.com/user-attachments/assets/ff9edcbc-2948-4f2a-a19b-14404249a6b6" />

## Privilege Escalation

With an initial shell established, I began Linux privilege escalation enumeration.

I searched the system for binaries owned by `root` with the SUID permission enabled.

```bash
find / -user root -perm -4000 -exec ls -ldb {} \;
```

Among the results, `/bin/systemctl` stood out as an unusual SUID binary.

```text
/bin/systemctl
```

Because `systemctl` was configured with SUID permissions, I investigated whether the misconfiguration could be abused to execute commands with elevated privileges.

I successfully exploited the SUID `systemctl` configuration and executed commands with root privileges.

<img width="615" height="25" alt="image" src="https://github.com/user-attachments/assets/779ff68a-d326-4b0e-8569-e9d10f34b698" />

After gaining root-level access, I located and retrieved the **root flag**.

<img width="266" height="25" alt="image" src="https://github.com/user-attachments/assets/5f989725-510a-44af-93df-33a5fc32970e" />

This completed the lab with full root access to the target system.
