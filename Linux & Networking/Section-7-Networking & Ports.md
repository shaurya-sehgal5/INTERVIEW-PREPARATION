# Networking & Ports — DevOps Interview Revision Guide

## What This README Covers

- Ports, sockets, listening services, TCP vs UDP
- `ss`, `lsof`, `curl`, `ping`, `nc`, `dig`, `nslookup`, `traceroute`, and `ip`
- `/etc/hosts`, `/etc/resolv.conf`, and `/etc/services`
- Real-world networking troubleshooting scenarios
- Interview questions with concise answers
- Production troubleshooting checklists
- Commands worth memorizing before an interview

## Quick Command Reference

| Goal | Command |
| --- | --- |
| Show all listening ports | `sudo ss -tulpn` |
| Find process using a port | `sudo lsof -i :8080` |
| Test HTTP service | `curl -I localhost:8080` |
| Test TCP port | `nc -zv host port` |
| Test basic connectivity | `ping host` |
| Check DNS | `dig +short domain.com` |
| Trace network path | `traceroute domain.com` |
| Show interfaces | `ip addr` |
| Show routing table | `ip route` |

> **Interview rule:** Start with the smallest useful test and move outward: **service → listening port → local connectivity → remote connectivity → firewall → DNS → network path**.

---

## Core Concepts

### What is a Port?

- A port is like a door on your server
- Applications listen on specific ports to receive traffic
- Common ports:
  - Port 22: SSH
  - Port 80: HTTP (web)
  - Port 443: HTTPS (secure web)
  - Port 5432: PostgreSQL
  - Port 3306: MySQL
  - Port 6379: Redis
  - Port 27017: MongoDB

### What is a Socket?

- IP Address + Port = Socket
- Example: `192.168.1.100:8080`
- This is how applications connect to each other

### TCP vs UDP

| Protocol | What it is | Examples |
| --- | --- | --- |
| **TCP** | Reliable, connection-based | HTTP, SSH, PostgreSQL |
| **UDP** | Faster, connectionless | DNS, video streaming |

**Interview Answer:**

> "TCP is reliable and ensures data arrives in order. UDP is faster but doesn't guarantee delivery. For most application traffic, we use TCP. For DNS or video streaming, UDP is preferred."

---

### What is Listening?

When a service is "listening" on a port:

- It's waiting for incoming connections
- Example: `nginx` listens on port 80
- Example: `sshd` listens on port 22

```
0.0.0.0:8080     → Listening on all interfaces (accessible from anywhere)
127.0.0.1:8080   → Listening only on localhost (only local access)
```

---

## Essential Commands

### 1. ss — Socket Statistics (Modern, Preferred)

```
# Show all listening ports
ss -tuln

# Show with process info (needs sudo)
sudo ss -tulpn

# Show all TCP connections
ss -t

# Show all UDP connections
ss -u

# Show all listening and non-listening sockets
ss -a

# Show only listening sockets
ss -l
```

**Options Breakdown:**

| OptionMeaning |                                       |
| ------------- | ------------------------------------- |
| `-t`          | TCP                                   |
| `-u`          | UDP                                   |
| `-l`          | Listening sockets only                |
| `-n`          | Show port numbers (not service names) |
| `-p`          | Show process using the socket         |

**Example Output:**

```
$ sudo ss -tulpn
Netid  State   Recv-Q  Send-Q  Local Address:Port    Peer Address:Port   Process
tcp    LISTEN  0       128     0.0.0.0:22             0.0.0.0:*           users:(("sshd",pid=800,fd=5))
tcp    LISTEN  0       128     0.0.0.0:80             0.0.0.0:*           users:(("nginx",pid=1200,fd=8))
tcp    LISTEN  0       128     0.0.0.0:5432           0.0.0.0:*           users:(("postgres",pid=900,fd=7))
tcp    LISTEN  0       128     [::]:8080              [::]:*              users:(("node",pid=1500,fd=18))
```

**What you see:**

- Port 22 → SSH (sshd)
- Port 80 → nginx (web server)
- Port 5432 → PostgreSQL
- Port 8080 → Node.js app

---

### 2. netstat — Older but Still Used

```
# Show all listening ports
netstat -tuln

# Show with process info
sudo netstat -tulpn

# Show all connections
netstat -an

# Show routing table
netstat -rn
```

**Note:** `ss` is newer and faster than `netstat`. Use `ss` in interviews to show you're modern. But know that `netstat` is still common in older systems.

---

### 3. lsof — List Open Files

Remember: **Linux treats everything as a file** — including network connections.

```
# Find which process is using port 8080
sudo lsof -i :8080

# Show all network connections
sudo lsof -i

# Show TCP connections only
sudo lsof -i tcp

# Show UDP connections only
sudo lsof -i udp

# Show connections for specific user
sudo lsof -i -u username
```

**Example Output:**

```
$ sudo lsof -i :8080
COMMAND  PID   USER   FD   TYPE  DEVICE SIZE/OFF NODE NAME
node    1500  app    18u  IPv4  12345      0t0  TCP *:8080 (LISTEN)
```

**What you see:**

- Process name: `node`
- PID: `1500`
- User: `app`
- Port: `8080`
- State: `LISTEN`

---

### 4. curl — Test HTTP/HTTPS Connections

```
# Test if a web service is responding
curl localhost:8080
curl http://192.168.1.100:8080

# Test external API
curl https://api.github.com

# Show response headers only
curl -I localhost:8080

# Follow redirects
curl -L google.com

# Test with timeout (important for troubleshooting)
curl --connect-timeout 5 localhost:8080

# Show detailed connection info
curl -v localhost:8080

# Save output to file
curl -o output.html http://example.com
```

**Interview scenario:**

> "How do you check if your application is responding?"

**Answer:**

```
curl localhost:8080
# or
curl -I localhost:8080   # Check headers/status code
```

---

### 5. ping — Basic ICMP Connectivity

```
# Check if host is reachable
ping google.com
ping 8.8.8.8

# Limited pings
ping -c 4 google.com

# Set interval
ping -i 0.5 google.com
```

**What to look for:**

- Packets sent/received
- Packet loss (should be 0%)
- Response time (should be low)

**Interview scenario:**

> "Your server can't reach the internet. How do you diagnose?"

**Answer:**

```
# Step 1: Ping external IP (bypass DNS)
ping 8.8.8.8

# Step 2: If that works, DNS might be the problem
ping google.com

# Step 3: Check DNS if domain fails
nslookup google.com
```

---

### 6. telnet / nc — Test Port Connectivity

```
# Test if a port is open/reachable
telnet google.com 80
telnet localhost 22

# Using netcat (nc)
nc -zv localhost 22
nc -zv 192.168.1.100 5432

# Send test data
echo "GET /" | nc google.com 80
```

**Options for nc:**

- `-z`: Test connection without sending data
- `-v`: Verbose output
- `-n`: No DNS resolution

**Interview scenario:**

> "How do you check if port 5432 is reachable from your application server to the database server?"

**Answer:**

```
nc -zv db-server 5432
# or
telnet db-server 5432
```

---

### 7. dig / nslookup — DNS Queries

```
# Check DNS resolution
dig google.com
nslookup google.com

# Query specific DNS server
dig @8.8.8.8 google.com

# Get only IP addresses
dig +short google.com

# Get specific record type
dig google.com MX
dig google.com CNAME

# Reverse DNS lookup
dig -x 8.8.8.8
```

**Interview scenario:**

> "Your application can't connect to [api.example.com](https://api.example.com/). How do you check if DNS is the problem?"

**Answer:**

```
# Step 1: Check DNS resolution
nslookup api.example.com

# Step 2: Check with external DNS
dig @8.8.8.8 api.example.com

# Step 3: Check hosts file
cat /etc/hosts
```

---

### 8. traceroute — Network Path Analysis

```
# Show the route to a destination
traceroute google.com

# Faster version (uses UDP)
traceroute -U google.com

# With IP only (no DNS resolution)
traceroute -n google.com

# TCP traceroute
traceroute -T google.com

# Limited hops
traceroute -m 10 google.com
```

**What it shows:**

- Each hop the packet takes
- Response time at each hop
- Where the connection fails

**Interview scenario:**

> "How do you find where the network is breaking?"

**Answer:**

```
traceroute google.com   # Shows each hop and where it fails
```

---

### 9. ip — Modern Network Configuration

```
# Show all network interfaces
ip addr
ip a

# Show routing table
ip route
ip r

# Show neighbor (ARP) table
ip neigh

# Bring interface up/down
sudo ip link set eth0 up
sudo ip link set eth0 down

# Add IP address
sudo ip addr add 192.168.1.100/24 dev eth0
```

**Note:** `ip` is replacing `ifconfig` on modern Linux.

---

## Important Files

### `/etc/hosts` — Local DNS Resolution

```
# View hosts file
cat /etc/hosts

# Example content
127.0.0.1   localhost
::1         localhost ip6-localhost ip6-loopback
192.168.1.10   myapp.local
```

**Why it matters:**

- Overrides DNS resolution
- Used for local development
- Sometimes used for service discovery

### `/etc/resolv.conf` — DNS Configuration

```
# View DNS settings
cat /etc/resolv.conf

# Example content
nameserver 8.8.8.8
nameserver 1.1.1.1
search mydomain.com
```

**Why it matters:**

- Tells your server which DNS servers to use
- Container networking issues often involve this file

### `/etc/services` — Common Ports Database

```
# See well-known ports
cat /etc/services | grep http
# Output: http  80/tcp

cat /etc/services | grep https
# Output: https  443/tcp

cat /etc/services | grep ssh
# Output: ssh  22/tcp
```

---

## Troubleshooting Scenarios

### Scenario 1: Port Already in Use

**Error:**

```
Error: listen EADDRINUSE: address already in use :::8080
```

**Solution:**

```
# Step 1: Find what's using port 8080
sudo ss -tulpn | grep :8080
# OR
sudo lsof -i :8080

# Step 2: Get the PID
# Output: node 1500 app user *:8080 (LISTEN)

# Step 3: Check the process
ps -fp 1500

# Step 4: Decide if you want to kill it
kill 1500        # Graceful
# OR
kill -9 1500     # Force

# Step 5: Start your application
npm start
# OR
sudo systemctl start myapp
```

**Interview Answer:**

> "First I'll check which process is using port 8080 with `ss -tulpn | grep :8080`. Then I'll verify if that process should be running. If it's a stale process, I'll kill it with `kill <PID>` and restart my application. If it's a legitimate service, I'll change my application's port."

---

### Scenario 2: Application Can't Connect to Database

**Error:**

```
Error: ECONNREFUSED 5432
```

**Solution:**

```
# Step 1: Check if database is running
sudo systemctl status postgresql

# Step 2: Check if it's listening on the right interface
sudo ss -tulpn | grep 5432
# Should show LISTEN, not just on localhost

# Step 3: Check connectivity from application server
nc -zv db-server 5432

# Step 4: Check if firewalls are blocking
# (AWS security groups, local iptables, etc.)

# Step 5: Check if PostgreSQL config allows remote connections
grep listen_addresses /etc/postgresql/*/main/postgresql.conf
```

---

### Scenario 3: Service Not Accessible from Outside

**Problem:**
Service runs locally but can't be reached from another machine.

**Solution:**

```
# Step 1: Check if it's listening on all interfaces
sudo ss -tulpn | grep 8080
# Look for: 0.0.0.0:8080 or *:8080 (good)
# If it shows: 127.0.0.1:8080 (bad - localhost only)

# Step 2: Check firewall
sudo iptables -L -n
# OR on AWS: Check security groups

# Step 3: Check if the service is actually running
curl localhost:8080

# Step 4: Check if port is open from another machine
nc -zv server-ip 8080
```

---

### Scenario 4: DNS Resolution Fails

**Problem:**
Application can't connect to `api.example.com`

**Solution:**

```
# Step 1: Check DNS resolution
nslookup api.example.com

# Step 2: Check with external DNS
dig @8.8.8.8 api.example.com

# Step 3: Check resolv.conf
cat /etc/resolv.conf

# Step 4: Check hosts file
cat /etc/hosts

# Step 5: Test connectivity to IP directly
ping $(dig +short api.example.com)
```

---

### Scenario 5: Connection Times Out

**Problem:**
Application times out when connecting to external service

**Solution:**

```
# Step 1: Check if host is reachable
ping external-service.com

# Step 2: Check if port is open
nc -zv external-service.com 443

# Step 3: Check route
traceroute external-service.com

# Step 4: Check firewall
sudo iptables -L -n

# Step 5: Check if port is blocked by cloud provider
# Check AWS Security Groups / Azure NSG / GCP Firewall
```

---

### Scenario 6: Check All Open Ports

```
# All listening ports with services
sudo ss -tulpn

# Count listening ports
sudo ss -tulpn | wc -l

# Show only TCP ports
sudo ss -tlnp

# Show only UDP ports
sudo ss -ulnp

# Show ports in use by specific process
sudo ss -tulpn | grep nginx
```

---

## Interview Questions & Answers

### Q1: Your application fails with "Port 8080 is already in use." What commands do you run?

**Answer:**

```
sudo ss -tulpn | grep :8080
# OR
sudo lsof -i :8080
```

### Q2: How do you check if a PostgreSQL server is listening on port 5432?

**Answer:**

```
sudo ss -tulpn | grep 5432
# OR
sudo netstat -tulpn | grep 5432
```

### Q3: What's the difference between `0.0.0.0:8080` and `127.0.0.1:8080`?

**Answer:**

> "`0.0.0.0` means listening on all network interfaces (accessible from outside). `127.0.0.1` means only localhost (only accessible from the same machine)."

### Q4: How do you check if your server can reach the internet?

**Answer:**

```
# Step 1: Test with IP first (bypass DNS)
ping 8.8.8.8

# Step 2: Test with domain (checks DNS)
ping google.com
```

### Q5: What command shows all listening ports with process info?

**Answer:**

```
sudo ss -tulpn
```

### Q6: How do you test if port 22 (SSH) is open on a remote server?

**Answer:**

```
nc -zv remote-server 22
# OR
telnet remote-server 22
```

### Q7: What's the difference between TCP and UDP?

**Answer:**

> "TCP is reliable and ensures data arrives in order with error checking. UDP is faster but doesn't guarantee delivery. Most application traffic (HTTP, SSH, databases) uses TCP. DNS uses UDP by default."

### Q8: How do you check DNS resolution?

**Answer:**

```
nslookup google.com
# OR
dig google.com
```

### Q9: How do you trace the route to a destination?

**Answer:**

```
traceroute google.com
```

### Q10: How do you check which process is using a specific port?

**Answer:**

```
sudo lsof -i :8080
# OR
sudo ss -tulpn | grep :8080
```

### Q11: How do you view all network interfaces?

**Answer:**

```
ip addr
# OR
ip a
```

### Q12: How do you check the routing table?

**Answer:**

```
ip route
# OR
netstat -rn
```

---

## Command Comparison Table

| Command | What it does | Common options |
| --- | --- | --- |
| `ss`                              | Show socket statistics              | `-tulpn` (show all TCP/UDP listening with processes) |
| `netstat`                         | Show network connections            | `-tulpn` (similar to ss)                             |
| `lsof`                            | List open files (including sockets) | `-i :port` (find process using port)                 |
| `curl`                            | Test HTTP/HTTPS connectivity        | `-I` (headers), `--connect-timeout`                  |
| `ping`                            | Test ICMP connectivity              | `-c` (count), `-i` (interval)                        |
| `telnet`                          | Test TCP port connectivity          | `<host> <port>`                                      |
| `nc`                              | Netcat - test ports                 | `-zv` (test connection)                              |
| `dig`                             | DNS queries                         | `+short` (brief output)                              |
| `nslookup`                        | DNS queries                         | `<domain>`                                           |
| `traceroute`                      | Show network path                   | `-n` (no DNS)                                        |
| `ip`                              | Network interface management        | `addr`, `route`, `link`                              |

---

## Quick Reference Card

### Find what's listening on a port

```
sudo ss -tulpn | grep :8080
sudo lsof -i :8080
```

### Check connectivity

```
ping google.com
curl localhost:8080
nc -zv db-server 5432
```

### DNS troubleshooting

```
nslookup google.com
dig +short google.com
cat /etc/resolv.conf
```

### Show all listening ports

```
sudo ss -tulpn
sudo netstat -tulpn
```

### Test network path

```
traceroute google.com
```

### Network interfaces

```
ip addr
ip route
```

---

## Commands to Memorize

### Must-Know Commands (Know These Cold!)

```
# Core networking commands
ss -tulpn                          # All listening ports with processes
sudo ss -tulpn                     # With process info
lsof -i :8080                      # Find process using specific port
curl localhost:8080                # Test HTTP connectivity
ping google.com                    # Basic connectivity
nc -zv host port                   # Test port connectivity
nslookup google.com                # DNS resolution
dig +short google.com              # Quick DNS
traceroute google.com              # Network path
ip addr                            # Network interfaces
ip route                           # Routing table
```

### Important Combinations

```
# Find and kill process on port
sudo lsof -i :8080
kill -9 PID

# Check if service is accessible
curl -I localhost:8080
curl --connect-timeout 5 localhost:8080

# DNS debugging
dig @8.8.8.8 example.com
host example.com
cat /etc/resolv.conf
cat /etc/hosts
```

---

## Production Troubleshooting Checklist

### When Application Can't Connect

1. **Is the service running?**

   bash

   Copy

   Download
   ```
   sudo systemctl status myapp
   ps aux | grep myapp
   ```
2. **Is it listening on the right port?**

   bash

   Copy

   Download
   ```
   sudo ss -tulpn | grep 8080
   ```
3. **Is it listening on the right interface?**
   - `0.0.0.0:8080` ✓ (accessible)
   - `127.0.0.1:8080` ✗ (localhost only)
4. **Can you connect locally?**

   bash

   Copy

   Download
   ```
   curl localhost:8080
   ```
5. **Can you connect remotely?**

   bash

   Copy

   Download
   ```
   nc -zv server-ip 8080
   ```
6. **Is firewall blocking?**

   bash

   Copy

   Download
   ```
   sudo iptables -L -n
   # Check cloud security groups
   ```
7. **Is DNS resolution working?**

   bash

   Copy

   Download
   ```
   nslookup api.example.com
   ```
8. **Is there a network path?**

   bash

   Copy

   Download
   ```
   traceroute api.example.com
   ```

### When Server Can't Reach Internet

1. **Check physical connection**

   bash

   Copy

   Download
   ```
   ip addr
   ```
2. **Test external IP (bypass DNS)**

   bash

   Copy

   Download
   ```
   ping 8.8.8.8
   ```
3. **If IP works, DNS is issue**

   bash

   Copy

   Download
   ```
   nslookup google.com
   cat /etc/resolv.conf
   ```
4. **Check default route**

   bash

   Copy

   Download
   ```
   ip route
   ```
5. **Check firewall**

   bash

   Copy

   Download
   ```
   sudo iptables -L -n
   ```

---

## Interview Tips

### What Interviewers Look For:

1. **Practical knowledge** — Can you actually use these commands?
2. **Logical troubleshooting** — Do you know the right order to check?
3. **Understanding** — Do you know why something works, not just what command to run?
4. **Security awareness** — Do you know the difference between `0.0.0.0` and `127.0.0.1`?

### Common Interview Traps:

1. **Using ****`netstat`**** when ****`ss`**** is more modern**
   - Do: `sudo ss -tulpn`
   - Don't: `netstat -tulpn` (though it still works)
2. **Not checking if port is listening on all interfaces**
   - Look for `0.0.0.0` or `*`
   - `127.0.0.1` means localhost only
3. **Killing processes without checking what they are**
   - Always check `ps -fp PID` before killing
4. **Not knowing DNS flow**
   - Application → `/etc/nsswitch.conf` → `/etc/hosts` → DNS → `/etc/resolv.conf`

---

## Key Takeaways

1. **Always use ****`ss -tulpn`** — It's the modern standard
2. **Check local first, then remote** — `curl localhost` then `nc -zv remote`
3. **DNS is common problem** — Check `nslookup` and `/etc/resolv.conf`
4. **Firewalls are common blocker** — Check `iptables` and cloud security groups
5. **`0.0.0.0`**** vs ****`127.0.0.1`** — Know the difference
6. **Find port conflicts with ****`lsof -i :port`**
7. **Test connectivity with ****`nc -zv`**

## Final Interview Checklist

Before an interview, make sure you can answer these without looking them up:

- [ ] What is a port?
- [ ] What is a socket?
- [ ] TCP vs UDP?
- [ ] `0.0.0.0` vs `127.0.0.1`?
- [ ] How do you find what is using port `8080`?
- [ ] How do you test whether a remote port is reachable?
- [ ] How do you troubleshoot DNS?
- [ ] How do you inspect the routing table?
- [ ] How do you identify where a network path is failing?
- [ ] What is the correct order for troubleshooting an unreachable application?

### Core Commands to Know Cold

```bash
sudo ss -tulpn
sudo lsof -i :8080
curl localhost:8080
curl -I localhost:8080
nc -zv host port
nslookup google.com
dig +short google.com
traceroute google.com
ip addr
ip route
```

---

**Section:** 7 — Networking & Ports  
**Purpose:** DevOps / Platform Engineer interview revision  
**Source:** User-provided networking study material
