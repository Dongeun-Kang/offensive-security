# Ignite

This is a lab for practicing web application enumeration, Remote Code Execution, and Linux privilege escalation.

The objective of this lab is to enumerate the target, identify a vulnerable web application, exploit the vulnerability to gain an initial shell, and escalate privileges to root.

Firstly, I performed basic enumeration against the target using RustScan and Nmap.

<img width="550" height="133" alt="image" src="https://github.com/user-attachments/assets/e6742949-0950-43d3-a42d-3cc7ee893c6b" />

The scan showed that port `80` was open, indicating that the target was running a web server.

I investigated the website running on port 80.

<img width="793" height="358" alt="image" src="https://github.com/user-attachments/assets/ba39bb88-0a08-4a86-b31b-c42a6f0f4d42" />

During enumeration, I identified that the website was running `Fuel CMS Version 1.4`.

I researched known vulnerabilities affecting this version of Fuel CMS.

<img width="652" height="262" alt="image" src="https://github.com/user-attachments/assets/5f1fb425-f23e-41b0-9e60-b9b0b4746db8" />

I found that Fuel CMS 1.4 was vulnerable to Remote Code Execution through `CVE-2018-16763`.

I downloaded a publicly available exploit script for the vulnerability and modified the required parameters for the target environment.

<img width="660" height="92" alt="image" src="https://github.com/user-attachments/assets/40165aad-bf7b-4836-8723-c56a661db63c" />

After executing the exploit, I successfully achieved Remote Code Execution on the target.

<img width="648" height="120" alt="image" src="https://github.com/user-attachments/assets/2058aadb-8d52-4155-9c94-435c661c6b26" />

With access to the target system, I enumerated the filesystem and located the user flag.

<img width="365" height="50" alt="image" src="https://github.com/user-attachments/assets/30eeefd4-28b6-466b-bc79-1a50b9a92440" />

After gaining initial access, I began privilege escalation enumeration.

While investigating the Fuel CMS configuration files, I found a `database.php` file containing database credentials stored in plaintext.

<img width="440" height="340" alt="image" src="https://github.com/user-attachments/assets/acb7c381-31e8-4ae7-ac91-d53e2e610efb" />

The exposed credentials included the password for the `root` account.

Before switching users, I upgraded the existing shell to a more stable interactive shell.

I then used the discovered credentials to authenticate as `root`.

<img width="816" height="76" alt="image" src="https://github.com/user-attachments/assets/b2ea1232-928c-42e4-a66f-9632dfe4fecf" />

After successfully gaining root privileges, I located and retrieved the root flag.

<img width="273" height="48" alt="image" src="https://github.com/user-attachments/assets/36b296f5-1fe5-433d-b593-104a39524be0" />

This completed the lab with full root access to the target system.
