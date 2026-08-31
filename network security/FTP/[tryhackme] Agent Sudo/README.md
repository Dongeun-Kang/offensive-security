# Agent Sudo

This is a lab for practicing network enumeration, web enumeration, credential attacks, file analysis, steganography, and password cracking.

The objective is to enumerate the target, identify hidden information in the web application, obtain credentials, investigate files retrieved from FTP, and eventually gain access to the target through SSH.

Firstly, I performed basic enumeration against the target using RustScan and Nmap.

<img width="391" height="320" alt="image" src="https://github.com/user-attachments/assets/a0d3c14b-62e1-4145-9f56-8f0f8b4e6beb" />

The scan showed that three ports were open on the target.

While investigating the website, I found a suspicious keyword related to the `User-Agent` header.

This made me think that the application might respond differently depending on the HTTP `User-Agent` value.

I intercepted the request using Burp Suite and started modifying the `User-Agent` header manually.

After testing several values, I found that using `R` produced a different response.

<img width="872" height="426" alt="image" src="https://github.com/user-attachments/assets/b2a96dab-728a-4307-8902-3673aafe4a4d" />

Since one alphabet character worked, I suspected that other characters might reveal additional information.

Instead of testing every character manually, I used Burp Suite Intruder to send requests containing different alphabet characters as the `User-Agent`.

Eventually, I discovered another page that revealed the name of one of the agents.

<img width="482" height="427" alt="image" src="https://github.com/user-attachments/assets/b0bc1b0d-5b27-49a9-9292-abacb00db086" />

I used the discovered agent name as a potential username and attempted to authenticate to the FTP service.

Since I did not know the password, I performed a password brute-force attack against the FTP account.

After obtaining the correct credentials, I successfully logged into the FTP server.

I found a text file along with several image files, including JPG and PNG files.

<img width="1261" height="132" alt="image" src="https://github.com/user-attachments/assets/2a441037-71c7-45f5-92f9-89595b3f7687" />

I downloaded the files and began investigating whether additional data was hidden inside the images.

I tested several file analysis and steganography tools.

Eventually, `binwalk` successfully detected embedded data inside one of the JPG files and extracted a ZIP archive.

I initially attempted to extract the archive using `unzip`, but it did not work correctly.

I then tried `7z` and discovered that the archive was protected by a passphrase.

To recover the passphrase, I generated a crackable hash from the protected file and used John the Ripper to perform password cracking.

After cracking the archive password, I extracted its contents and continued investigating the recovered files.

This eventually revealed credentials that could be used for SSH authentication.

<img width="300" height="31" alt="image" src="https://github.com/user-attachments/assets/54a8ae3e-1917-420e-8ca1-8971bc72a86d" />

With the recovered credentials, I was able to proceed with SSH access to the target machine.
