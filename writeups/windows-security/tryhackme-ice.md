# Ice

## Reconnaissance

Started with **RustScan** and **Nmap** to identify open ports and running services on the target.

<img width="1007" height="910" alt="image" src="https://github.com/user-attachments/assets/a2424d83-5344-4dfa-959d-7cfa86476288" />

The scan revealed an **Icecast** service running on the target.

---

## Vulnerability Research

Searched for known vulnerabilities affecting the detected Icecast service and identified a potentially exploitable vulnerability.

<img width="1127" height="280" alt="image" src="https://github.com/user-attachments/assets/b351070e-7005-416e-ad9a-bf49107273e3" />

---

## Initial Access

Used **Metasploit** to exploit the vulnerable Icecast service and successfully gained an initial shell on the target.

<img width="505" height="142" alt="image" src="https://github.com/user-attachments/assets/1f361393-9b85-45dc-91c2-f0f398e0b6e3" />

---

## Privilege Escalation

Performed internal enumeration through the existing Metasploit session to identify possible privilege escalation vectors.

<img width="1252" height="288" alt="image" src="https://github.com/user-attachments/assets/4daaa447-7211-4c2b-8158-2004c7bab973" />

The enumeration identified another vulnerability that could potentially be exploited for privilege escalation.

After exploiting it, a new shell with elevated privileges was obtained.

<img width="872" height="197" alt="image" src="https://github.com/user-attachments/assets/48201062-d184-4fec-8123-5d0069549e76" />

---

## Post-Exploitation / Looting

Migrated the Meterpreter session to a more stable process with higher privileges.

<img width="300" height="75" alt="image" src="https://github.com/user-attachments/assets/6fd1aa7c-4667-4e80-9dde-85bf087d8f44" />

Successfully obtained:

```text
NT AUTHORITY\SYSTEM
```

With **SYSTEM-level privileges**, loaded the `kiwi` extension in Meterpreter and extracted credentials from the compromised Windows host.

<img width="1016" height="462" alt="image" src="https://github.com/user-attachments/assets/32172db7-ce1b-43f2-a563-ce20d666119d" />

---

## Attack Path

```text
RustScan / Nmap
        ↓
Icecast Service Discovery
        ↓
Vulnerability Research
        ↓
Metasploit Exploitation
        ↓
Initial Access
        ↓
Internal Enumeration
        ↓
Privilege Escalation
        ↓
NT AUTHORITY\SYSTEM
        ↓
Credential Dumping with Kiwi
```

## Key Takeaways

* Practiced service enumeration using **RustScan** and **Nmap**.
* Identified and exploited a vulnerable **Icecast** service using **Metasploit**.
* Performed Windows privilege escalation and obtained **NT AUTHORITY\SYSTEM** privileges.
* Used Meterpreter post-exploitation capabilities such as **process migration** and **Kiwi credential extraction**.
