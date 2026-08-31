# Agent Sudo

This is a lab for practicing network enumeration, web enumeration, credential attacks, file analysis, steganography, password cracking, and Linux privilege escalation.

The objective is to enumerate the target, discover hidden information through HTTP requests, obtain credentials, gain access through SSH, and escalate privileges to root.

Firstly, I performed basic enumeration against the target using RustScan and Nmap.

<img width="391" height="320" alt="image" src="https://github.com/user-attachments/assets/a0d3c14b-62e1-4145-9f56-8f0f8b4e6beb" />

The scan showed that three ports were open on the target.

While investigating the website, I found a suspicious keyword related to the `User-Agent` header.

This made me think that the application might respond differently depending on the HTTP `User-Agent` value.

I intercepted the request using Burp Suite and manually modified the `User-Agent` header.

After testing several values, I found that using `R` produced a different response.

<img width="872" height="426" alt="image" src="https://github.com/user-attachments/assets/b2a96dab-728a-4307-8902-3673aafe4a4d" />

Since one alphabet character worked, I suspected that other characters could reveal additional information.

Instead of testing every character manually, I used Burp Suite Intruder to send requests containing different alphabet characters as the `User-Agent`.

Eventually, I discovered another page that revealed the name of one of the agents.

<img width="482" height="427" alt="image" src="https://github.com/user-attachments/assets/b0bc1b0d-5b27-49a9-9292-abacb00db086" />

I used the discovered agent name as a potential username and attempted to authenticate to the FTP service.

Since I did not know the password, I brute-forced the FTP account and successfully obtained valid credentials.

After logging into the FTP server, I found a text file and several image files.

<img width="1261" height="132" alt="image" src="https://github.com/user-attachments/assets/2a441037-71c7-45f5-92f9-89595b3f7687" />

I downloaded the files and investigated whether additional information was hidden inside the images.

After testing several tools, I found that `binwalk` successfully detected embedded data inside one of the JPG files and extracted a ZIP archive.

I initially attempted to extract the archive using `unzip`, but it did not work.

I then used `7z` and discovered that the archive was protected by a passphrase.

I generated a crackable hash from the protected file and used John the Ripper to recover the password.

After extracting the archive and investigating its contents, I eventually recovered credentials that could be used for SSH authentication.

I then logged into the target through SSH and successfully obtained the user flag.

<img width="300" height="31" alt="image" src="https://github.com/user-attachments/assets/54a8ae3e-1917-420e-8ca1-8971bc72a86d" />

After gaining access as the user, I performed further enumeration to identify a privilege escalation path.

I discovered a way to abuse the user's sudo privileges and escalate my privileges to root.

After successfully gaining root access, I located and retrieved the root flag.

<img width="946" height="322" alt="image" src="https://github.com/user-attachments/assets/71ae5d46-302a-4adf-8806-ddb6124abe81" />

This completed the lab with full root access to the target system.
