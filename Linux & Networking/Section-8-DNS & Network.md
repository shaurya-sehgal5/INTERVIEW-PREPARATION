# DNS & Network Troubleshooting

> Complete revision guide for **DevOps / Platform Engineer interviews**
> **Focus:** DNS concepts, resolution flow, troubleshooting, essential commands, and interview-ready answers.

---

## 📚 Table of Contents

1. [Core Concepts](#1-core-concepts)
2. [DNS Resolution Flow](#2-dns-resolution-flow)
3. [Essential Commands](#3-essential-commands)
4. [Important Files](#4-important-files)
5. [DNS Cache](#5-dns-cache)
6. [Troubleshooting Scenarios](#6-troubleshooting-scenarios)
7. [Interview Questions & Answers](#7-interview-questions--answers)
8. [Quick Reference Card](#8-quick-reference-card)
9. [Production Troubleshooting Checklist](#9-production-troubleshooting-checklist)
10. [Common Interview Traps](#10-common-interview-traps)
11. [Key Takeaways](#11-key-takeaways)
12. [Commands to Memorize](#12-commands-to-memorize)

---

# 1. Core Concepts

## What is DNS?

**DNS (Domain Name System)** translates human-readable domain names into IP addresses.

```text
google.com → 142.250.185.78
```

### Why DNS Matters

* Humans remember names more easily than IP addresses.
* Computers communicate using IP addresses.
* DNS allows applications to use domain names instead of hard-coded IP addresses.
* DNS failures can prevent applications from connecting to services.

---

## DNS Record Types

| Record    | Purpose                                             | Example                               |
| --------- | --------------------------------------------------- | ------------------------------------- |
| **A**     | Maps domain to IPv4 address                         | `google.com → 142.250.185.78`         |
| **AAAA**  | Maps domain to IPv6 address                         | `google.com → 2001:4860:4860::8888`   |
| **CNAME** | Alias to another domain                             | `www.google.com → google.com`         |
| **MX**    | Mail exchange servers                               | `gmail.com → mail server`             |
| **TXT**   | Text records used for verification, SPF, DKIM, etc. | `v=spf1 include:_spf.google.com ~all` |
| **NS**    | Name servers responsible for a domain               | `ns1.google.com`                      |
| **PTR**   | Reverse DNS: IP → domain                            | `IP → hostname`                       |

### Interview Answer

> A records map domain names to IPv4 addresses. CNAME records create aliases to other domains, while MX records handle email routing.

---

# 2. DNS Resolution Flow

## High-Level Resolution Flow

```text
Application
    ↓
/etc/nsswitch.conf
    ↓
/etc/hosts
    ↓
DNS Query
    ↓
/etc/resolv.conf
    ↓
Recursive DNS Server
    ↓
Root DNS Servers
    ↓
TLD DNS Servers
    ↓
Authoritative DNS Server
    ↓
IP Address Returned
    ↓
Application Connects
```

---

## What Happens When You Type `google.com`?

```text
1. Application calls getaddrinfo("google.com")
        ↓
2. Check /etc/nsswitch.conf
   → hosts: files dns
        ↓
3. Check /etc/hosts
   → If found, use that IP
        ↓
4. If not found, query DNS
        ↓
5. Check /etc/resolv.conf
   → Find configured nameserver
        ↓
6. Send query to recursive resolver
        ↓
7. Recursive resolver contacts Root DNS Server
        ↓
8. Root server points to .com TLD server
        ↓
9. TLD server points to authoritative server
        ↓
10. Recursive resolver contacts authoritative server
        ↓
11. Authoritative server returns IP address
        ↓
12. Resolver returns the answer to the client
        ↓
13. Application receives the IP
        ↓
14. Connection is established
```

---

## Types of DNS Servers

| Server                        | Function                                                                       |
| ----------------------------- | ------------------------------------------------------------------------------ |
| **Recursive Resolver**        | Receives queries from clients and queries other DNS servers to find the answer |
| **Root Name Server**          | Directs queries to the appropriate TLD servers                                 |
| **TLD Name Server**           | Handles domains under TLDs such as `.com`, `.org`, `.in`                       |
| **Authoritative Name Server** | Contains the actual DNS records for a domain                                   |

---

# 3. Essential Commands

## `dig` — Most Powerful DNS Lookup Tool

### Basic Lookup

```bash
dig google.com
```

### Get Only IP Addresses

```bash
dig +short google.com
```

### Query Specific Record Types

```bash
dig google.com A
dig google.com AAAA
dig google.com CNAME
dig google.com MX
dig google.com NS
dig google.com TXT
```

### Query a Specific DNS Server

```bash
dig @8.8.8.8 google.com
```

### Reverse DNS Lookup

```bash
dig -x 142.250.185.78
```

### Show Only Answer Section

```bash
dig google.com +noall +answer
```

### Trace DNS Resolution

```bash
dig +trace google.com
```

### Show TTL

```bash
dig google.com +ttlid
```

---

## Understanding `dig` Output

Example:

```text
$ dig google.com

; <<>> DiG 9.11.4 <<>> google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 12345
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; QUESTION SECTION:
;google.com.                    IN      A

;; ANSWER SECTION:
google.com.             300     IN      A       142.250.185.78

;; Query time: 12 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
```

### Important Fields

| Field              | Meaning                   |
| ------------------ | ------------------------- |
| **QUESTION**       | What was requested        |
| **ANSWER**         | DNS response              |
| **300**            | TTL in seconds            |
| **142.250.185.78** | Returned IP address       |
| **Query time**     | DNS response time         |
| **SERVER**         | DNS server that responded |

---

## `nslookup` — Simple DNS Query

```bash
# Basic lookup
nslookup google.com

# Specific record types
nslookup -type=A google.com
nslookup -type=CNAME google.com
nslookup -type=MX google.com
nslookup -type=NS google.com

# Specific DNS server
nslookup google.com 8.8.8.8

# Reverse DNS
nslookup 142.250.185.78
```

### Interactive Mode

```bash
nslookup

> server 8.8.8.8
> google.com
> exit
```

---

## `host` — Simple DNS Lookup

```bash
# Basic lookup
host google.com

# Specific record type
host -t A google.com
host -t CNAME google.com
host -t MX google.com
host -t NS google.com

# Specific DNS server
host google.com 8.8.8.8

# Reverse DNS
host 142.250.185.78
```

---

## `whois` — Domain Information

```bash
# Domain registration information
whois google.com

# IP ownership information
whois 142.250.185.78

# Filter useful fields
whois google.com | grep -i "registrant\|creation\|expiration"
```

---

## DNS Cache Commands

### systemd-resolved

```bash
# Check status
systemd-resolve --status

# Flush DNS cache
sudo systemd-resolve --flush-caches

# Check cache statistics
systemd-resolve --statistics
```

### nscd

```bash
sudo /etc/init.d/nscd restart
```

### dnsmasq

```bash
sudo systemctl restart dnsmasq
```

### Windows

```powershell
ipconfig /flushdns
```

### macOS

```bash
sudo dscacheutil -flushcache
sudo killall -HUP mDNSResponder
```

---

# 4. Important Files

## `/etc/hosts`

### Purpose

Provides manual hostname-to-IP mappings and can override DNS depending on the configuration in `/etc/nsswitch.conf`.

```bash
cat /etc/hosts
```

Example:

```text
127.0.0.1       localhost
::1             localhost ip6-localhost
192.168.1.10    myapp.local
192.168.1.20    database.local
```

### Common Uses

* Local development
* Internal service discovery
* Testing without DNS
* Temporary hostname overrides
* Blocking domains

### Syntax

```text
<IP>    <hostname>    <alias1> <alias2>
```

---

## `/etc/resolv.conf`

### Purpose

Defines which DNS servers the system uses.

```bash
cat /etc/resolv.conf
```

Example:

```text
nameserver 8.8.8.8
nameserver 1.1.1.1
search mydomain.com
domain mydomain.com
options timeout:2 attempts:3
```

### Important Fields

| Field        | Purpose                             |
| ------------ | ----------------------------------- |
| `nameserver` | DNS server IP                       |
| `search`     | Domain suffix for unqualified names |
| `domain`     | Default domain                      |
| `options`    | Timeout and retry settings          |

On systems using `systemd-resolved`, `/etc/resolv.conf` may be a symlink:

```text
/etc/resolv.conf
    ↓
/run/systemd/resolve/stub-resolv.conf
```

---

## `/etc/nsswitch.conf`

### Purpose

Controls the order in which name-resolution sources are consulted.

```bash
cat /etc/nsswitch.conf
```

Relevant configuration:

```text
hosts: files dns myhostname
```

This means:

```text
1. /etc/hosts
       ↓
2. DNS
       ↓
3. Local hostname resolution
```

---

## `/etc/hostname`

Contains the system hostname.

```bash
cat /etc/hostname
```

Change the hostname:

```bash
sudo hostnamectl set-hostname myserver
```

---

# 5. DNS Cache

## What is DNS Cache?

DNS caching temporarily stores DNS responses.

Benefits:

* Faster subsequent queries
* Reduced DNS traffic
* Reduced load on DNS servers

The **TTL (Time To Live)** determines how long a record can remain cached.

---

## Cache Layers

```text
Application
    ↓
OS DNS Cache
    ↓
Local DNS Resolver
    ↓
ISP DNS Cache
    ↓
Authoritative DNS Server
```

---

# 6. Troubleshooting Scenarios

## Scenario 1 — Domain Not Resolving

### Problem

```text
Application cannot connect to:
api.example.com
```

### Diagnosis

```bash
# 1. Test DNS resolution
dig api.example.com

# 2. Test different DNS servers
dig @8.8.8.8 api.example.com
dig @1.1.1.1 api.example.com

# 3. Check domain information
whois example.com

# 4. Check DNS configuration
cat /etc/resolv.conf

# 5. Check hosts file
grep example.com /etc/hosts

# 6. Check DNS server connectivity
ping 8.8.8.8
```

### Possible Causes

* Domain does not exist
* DNS server is unavailable
* Network connectivity issue
* Incorrect DNS server configuration
* Incorrect `/etc/hosts` entry
* Stale DNS cache
* Firewall blocking DNS traffic on port `53`

---

## Scenario 2 — DNS Resolution is Slow

### Diagnosis

```bash
# Check DNS response time
dig google.com | grep "Query time"

# Compare DNS servers
dig @8.8.8.8 google.com | grep "Query time"
dig @1.1.1.1 google.com | grep "Query time"

# Check connectivity
ping 8.8.8.8

# Check timeout configuration
cat /etc/resolv.conf | grep timeout
```

### Possible Solutions

* Use a faster DNS server
* Reduce DNS timeout
* Use local DNS caching
* Investigate network latency

---

## Scenario 3 — Intermittent DNS Failures

### Problem

DNS works sometimes but fails at other times.

### Diagnosis

```bash
# Check DNS server health
ping 8.8.8.8 -c 10

# Test multiple DNS servers
dig @8.8.8.8 google.com
dig @1.1.1.1 google.com

# Check packet loss
ping -c 10 8.8.8.8 | grep loss

# Check systemd-resolved logs
sudo journalctl -u systemd-resolved -n 50
```

### Possible Solutions

* Configure multiple DNS servers
* Check network connectivity
* Monitor DNS server load
* Use a local caching resolver

---

## Scenario 4 — DNS Poisoning / Spoofing

### Problem

A domain resolves to an unexpected IP address.

### Diagnosis

```bash
# Compare different DNS sources
dig google.com
dig @8.8.8.8 google.com

# Check authoritative resolution
dig +trace google.com

# Check local hosts file
cat /etc/hosts

# Check resolver statistics
systemd-resolve --statistics
```

### Possible Solutions

* Use DNSSEC
* Use trusted DNS resolvers
* Verify TLS certificates

---

## Scenario 5 — Container DNS Issues

### Problem

Docker or Kubernetes containers cannot resolve domains.

### Docker

```bash
docker exec -it container -- cat /etc/resolv.conf
```

### Kubernetes

```bash
# Test DNS from a pod
kubectl exec -it pod -- nslookup google.com

# Check Kubernetes DNS service
kubectl get svc -n kube-system kube-dns

# Check CoreDNS logs
kubectl logs -n kube-system kube-dns-xxx

# Check CoreDNS configuration
kubectl get configmap coredns -n kube-system -o yaml
```

### Common Causes

* Kubernetes DNS service is not running
* NetworkPolicy blocks DNS traffic
* Incorrect DNS configuration inside the pod
* CoreDNS configuration problem

---

## Scenario 6 — Email Delivery Issues

### Check MX Records

```bash
dig example.com MX
```

### Check SPF

```bash
dig example.com TXT | grep spf
```

### Check DKIM

```bash
dig example.com TXT | grep dkim
```

### Check DMARC

```bash
dig _dmarc.example.com TXT
```

### Important Records

| Record    | Purpose                               |
| --------- | ------------------------------------- |
| **MX**    | Mail servers                          |
| **SPF**   | Sender authentication / anti-spoofing |
| **DKIM**  | Email signing                         |
| **DMARC** | Email authentication policy           |

---

# 7. Interview Questions & Answers

## Q1. What is DNS and why is it important?

**Answer:**

> DNS translates human-readable domain names to IP addresses. It enables users and applications to access services using names instead of remembering IP addresses.

---

## Q2. How do you check DNS resolution from the command line?

```bash
dig google.com
nslookup google.com
host google.com
```

---

## Q3. Difference between `dig` and `nslookup`?

> `dig` provides more detailed DNS information such as TTL, query time, and response sections. `nslookup` is simpler and useful for basic DNS queries. `dig` is generally preferred for detailed DevOps troubleshooting.

---

## Q4. How do you check which DNS server your system is using?

```bash
cat /etc/resolv.conf
```

Or:

```bash
systemd-resolve --status | grep -i "DNS Server"
```

---

## Q5. What is the DNS resolution order?

> The order is controlled by `/etc/nsswitch.conf`. A typical configuration checks `/etc/hosts` first and then DNS using the nameservers configured through `/etc/resolv.conf`.

---

## Q6. How do you flush DNS cache?

```bash
sudo systemd-resolve --flush-caches
```

Or:

```bash
sudo /etc/init.d/nscd restart
```

---

## Q7. What is TTL?

> TTL (Time To Live) determines how long a DNS record can be cached. A lower TTL allows DNS changes to propagate faster but increases DNS queries. A higher TTL reduces DNS traffic but can keep old records cached longer.

---

## Q8. How do you check MX records?

```bash
dig example.com MX
```

Or:

```bash
nslookup -type=MX example.com
```

---

## Q9. What is a CNAME record?

> A CNAME creates an alias pointing one domain name to another domain name.

Example:

```text
www.example.com → example.com
```

---

## Q10. How do you test whether a DNS server is working?

```bash
dig @8.8.8.8 google.com
```

Or:

```bash
nslookup google.com 8.8.8.8
```

---

## Q11. What is split-horizon DNS?

> Split-horizon DNS provides different DNS responses depending on where the request originates.

Example:

```text
Internal users → Private IP
External users → Public IP
```

---

## Q12. How do you troubleshoot DNS issues in Kubernetes?

```bash
# Check DNS service
kubectl get svc -n kube-system kube-dns

# Check pod DNS configuration
kubectl exec -it pod -- cat /etc/resolv.conf

# Test DNS from pod
kubectl exec -it pod -- nslookup google.com

# Check CoreDNS logs
kubectl logs -n kube-system kube-dns-xxx
```

---

## Q13. Authoritative vs Recursive DNS?

> Authoritative DNS servers contain the actual DNS records for domains. Recursive DNS servers query other DNS servers on behalf of clients and return the final answer.

---

## Q14. What is reverse DNS?

> Reverse DNS maps an IP address back to a domain or hostname. It is commonly used for verification and security-related checks.

```bash
dig -x 142.250.185.78
```

---

## Q15. How do you track changes to DNS records?

```bash
# Check current record
dig example.com A

# Check SOA information
dig example.com SOA
```

The SOA record contains a serial value that can be used to track DNS-zone changes.

---

# 8. Quick Reference Card

## Common DNS Commands

| Task                | Command                   |
| ------------------- | ------------------------- |
| Basic lookup        | `dig domain.com`          |
| Only IP             | `dig +short domain.com`   |
| Specific DNS server | `dig @8.8.8.8 domain.com` |
| A record            | `dig domain.com A`        |
| MX record           | `dig domain.com MX`       |
| CNAME record        | `dig domain.com CNAME`    |
| NS record           | `dig domain.com NS`       |
| TXT record          | `dig domain.com TXT`      |
| Reverse DNS         | `dig -x IP`               |
| DNS trace           | `dig +trace domain.com`   |
| TTL                 | `dig domain.com +ttlid`   |
| Simple lookup       | `nslookup domain.com`     |
| Host lookup         | `host domain.com`         |
| Domain information  | `whois domain.com`        |

---

## Important DNS Files

| File                 | Purpose                    |
| -------------------- | -------------------------- |
| `/etc/hosts`         | Local hostname/IP mappings |
| `/etc/resolv.conf`   | DNS resolver configuration |
| `/etc/nsswitch.conf` | Name-resolution order      |
| `/etc/hostname`      | System hostname            |

---

## DNS Cache Commands

| Task                | Command                               |
| ------------------- | ------------------------------------- |
| Flush systemd cache | `sudo systemd-resolve --flush-caches` |
| Cache statistics    | `systemd-resolve --statistics`        |
| DNS status          | `systemd-resolve --status`            |

---

## Troubleshooting Checklist

```text
1. Test DNS resolution
   ↓
   dig domain.com

2. Test another DNS server
   ↓
   dig @8.8.8.8 domain.com

3. Check /etc/hosts
   ↓
   cat /etc/hosts

4. Check DNS configuration
   ↓
   cat /etc/resolv.conf

5. Check DNS server connectivity
   ↓
   ping 8.8.8.8

6. Check DNS cache
   ↓
   Flush DNS cache

7. Check network path
   ↓
   traceroute 8.8.8.8

8. Check firewall rules
   ↓
   sudo iptables -L -n | grep 53
```

---

# 9. Production Troubleshooting Checklist

## DNS Not Resolving

```text
1. Is the DNS server reachable?
   → ping 8.8.8.8

2. Is the DNS service running?
   → systemctl status systemd-resolved

3. What DNS server is configured?
   → cat /etc/resolv.conf

4. Is the domain present in /etc/hosts?
   → grep domain.com /etc/hosts

5. Does the domain exist?
   → whois domain.com

6. Is the DNS server responding?
   → dig @8.8.8.8 domain.com

7. Could the DNS cache be stale?
   → sudo systemd-resolve --flush-caches

8. Is DNS traffic reaching the server?
   → tcpdump -i eth0 port 53
```

---

## DNS is Slow

```text
1. Check query time
   → dig domain.com | grep "Query time"

2. Compare DNS servers
   → dig @8.8.8.8 domain.com
   → dig @1.1.1.1 domain.com

3. Check network latency
   → ping 8.8.8.8

4. Check DNS server load
   → Monitor server metrics

5. Use local caching
   → systemd-resolved or dnsmasq
```

---

## Internal vs External DNS

### Internal DNS

* Runs inside a private/corporate network
* Resolves internal services
* Commonly returns private IP addresses

Example:

```text
app.internal.com → 10.x.x.x
```

### External DNS

* Publicly accessible DNS
* Resolves public domains
* Commonly managed through DNS providers

---

# 10. Common Interview Traps

## Trap 1 — Wrong Resolution Order

❌ **Bad Answer:**

> DNS is checked first.

✅ **Good Answer:**

> The system checks `/etc/hosts` first when configured through `/etc/nsswitch.conf`, then performs DNS resolution.

---

## Trap 2 — Not Understanding TTL

❌ **Bad Answer:**

> TTL is just a number.

✅ **Good Answer:**

> TTL determines how long DNS records remain cached. Lower TTLs allow faster changes but increase DNS queries, while higher TTLs reduce queries but delay changes.

---

## Trap 3 — Forgetting DNS Cache

❌ **Bad Answer:**

> The DNS record is wrong.

✅ **Good Answer:**

> First check whether the issue is caused by stale DNS cache by flushing the cache or querying another DNS server.

---

## Trap 4 — Ignoring Network Connectivity

❌ **Bad Answer:**

> DNS is broken.

✅ **Good Answer:**

> First verify that the DNS server is reachable, then test DNS resolution itself.

---

# 11. Key Takeaways

1. **DNS translates domain names to IP addresses.**
2. Typical resolution order is `/etc/hosts → DNS`.
3. Know the major records: **A, AAAA, CNAME, MX, TXT, NS, PTR**.
4. **TTL controls DNS caching duration.**
5. `dig` is the primary troubleshooting tool.
6. DNS should be checked when connectivity fails due to name resolution.
7. DNS cache can cause stale responses.
8. Firewalls can block DNS traffic on **port 53**.
9. Kubernetes uses DNS services such as **CoreDNS**.
10. Split-horizon DNS can return different answers for internal and external clients.

---

# 12. Commands to Memorize

## DNS Lookup

```bash
dig domain.com
dig +short domain.com
dig @8.8.8.8 domain.com
```

## Record Types

```bash
dig domain.com A
dig domain.com MX
dig domain.com CNAME
dig domain.com NS
dig domain.com TXT
```

## Reverse DNS

```bash
dig -x IP
```

## Trace DNS

```bash
dig +trace domain.com
```

## Other Tools

```bash
nslookup domain.com
host domain.com
whois domain.com
```

## DNS Configuration

```bash
cat /etc/hosts
cat /etc/resolv.conf
cat /etc/nsswitch.conf
```

## DNS Cache

```bash
sudo systemd-resolve --flush-caches
systemd-resolve --statistics
systemd-resolve --status
```

---

## 🎯 Interview Focus

For a **DevOps / Platform Engineer interview**, make sure you can explain without hesitation:

```text
DNS
 ├── What DNS does
 ├── A / AAAA / CNAME / MX / TXT / NS / PTR
 ├── Resolution flow
 ├── /etc/hosts
 ├── /etc/resolv.conf
 ├── /etc/nsswitch.conf
 ├── Recursive vs Authoritative DNS
 ├── TTL and caching
 ├── dig
 ├── DNS troubleshooting
 ├── Kubernetes DNS / CoreDNS
 └── Split-horizon DNS
```

**Most important commands:**

```bash
dig domain.com
dig +short domain.com
dig @8.8.8.8 domain.com
dig domain.com A
dig domain.com MX
dig domain.com CNAME
dig domain.com NS
dig domain.com TXT
dig -x IP
dig +trace domain.com
nslookup domain.com
host domain.com
cat /etc/hosts
cat /etc/resolv.conf
cat /etc/nsswitch.conf
```
