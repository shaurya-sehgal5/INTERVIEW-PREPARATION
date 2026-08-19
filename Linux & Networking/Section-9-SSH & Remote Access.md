# SSH & Remote Access

> Complete revision guide for **DevOps / Platform Engineer interviews**
> **Focus:** Practical SSH usage, key management, remote access, port forwarding, troubleshooting, and interview-ready answers.

---

## 📚 Table of Contents

1. [Core Concepts](#1-core-concepts)
2. [Essential Commands](#2-essential-commands)
3. [SSH Key Management](#3-ssh-key-management)
4. [Important Files](#4-important-files)
5. [SSH Config File](#5-ssh-config-file)
6. [Port Forwarding](#6-port-forwarding)
7. [Troubleshooting Scenarios](#7-troubleshooting-scenarios)
8. [Interview Questions & Answers](#8-interview-questions--answers)
9. [Quick Reference Card](#9-quick-reference-card)
10. [Production Troubleshooting Checklist](#10-production-troubleshooting-checklist)
11. [Security Best Practices](#11-security-best-practices)
12. [Common Interview Traps](#12-common-interview-traps)
13. [Key Takeaways](#13-key-takeaways)
14. [Commands to Memorize](#14-commands-to-memorize)

---

# 1. Core Concepts

## What is SSH?

**SSH (Secure Shell)** is a protocol used to securely connect to and administer remote systems.

```text
Local Machine
      │
      │ SSH
      ▼
Remote Server
(EC2 / VM / Linux Server)
```

### Why SSH Matters

SSH allows you to:

* Securely access remote servers
* Execute commands remotely
* Transfer files securely
* Tunnel traffic through an encrypted connection
* Perform remote administration

### Less Secure Alternatives

* Telnet — sends traffic in plain text
* Rlogin — no encryption
* FTP — traditionally unencrypted

### Interview Answer

> SSH is a secure protocol for remote administration. It encrypts traffic between the client and server, protecting communication from eavesdropping and unauthorized interception.

---

## SSH vs SSL/TLS

| Feature             | SSH                                 | SSL/TLS                          |
| ------------------- | ----------------------------------- | -------------------------------- |
| **Primary purpose** | Remote access and command execution | Secure network/web communication |
| **Common port**     | `22`                                | `443`                            |
| **Authentication**  | Passwords, keys, certificates       | Primarily certificates           |
| **Typical use**     | Server administration               | HTTPS                            |

---

## SSH Components

```text
┌─────────────────────────────────────────────────────────────┐
│                     SSH Connection                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Client Machine                 Remote Server               │
│                                                             │
│  ├── ssh command                ├── sshd daemon             │
│  ├── SSH private key            ├── /etc/ssh/sshd_config    │
│  └── ~/.ssh/config              └── ~/.ssh/authorized_keys │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

# 2. Essential Commands

## Basic SSH Connection

```bash
# Connect to remote server
ssh user@hostname

# Connect using a specific port
ssh -p 2222 user@hostname

# Connect as root
ssh root@hostname

# Connect using a specific identity file
ssh -i ~/.ssh/mykey.pem user@hostname

# Execute a remote command
ssh user@hostname "ls -la /var/log"

# Execute multiple commands
ssh user@hostname "cd /var/log && tail -n 50 app.log"

# Verbose debugging
ssh -v user@hostname
ssh -vv user@hostname
ssh -vvv user@hostname
```

### SSH Verbosity

| Option | Purpose                       |
| ------ | ----------------------------- |
| `-v`   | Verbose                       |
| `-vv`  | More verbose                  |
| `-vvv` | Maximum debugging information |

Use `ssh -v` when troubleshooting authentication or connection problems.

---

# 3. SSH Key Management

## SSH Key Pair

SSH key authentication uses two keys:

```text
Private Key
    │
    │ Keep secret
    ▼
Your Machine

Public Key
    │
    │ Copy to server
    ▼
~/.ssh/authorized_keys
```

### Important Rule

**Never share your private key.**

---

## Generate an RSA Key

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

Generate with a custom filename:

```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/mykey
```

---

## Generate an ED25519 Key

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

ED25519 is the recommended modern key type in this guide.

---

## View Public Key

```bash
cat ~/.ssh/id_ed25519.pub
```

For RSA:

```bash
cat ~/.ssh/id_rsa.pub
```

---

## Check Key Fingerprint

```bash
ssh-keygen -l -f ~/.ssh/id_rsa.pub
```

---

## Copy Public Key to Server

```bash
ssh-copy-id user@hostname
```

Or manually:

```bash
cat ~/.ssh/id_ed25519.pub | ssh user@hostname \
"mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

---

## SSH Key Types

| Type        |   Key Size | Security  | Speed  | Typical Usage   |
| ----------- | ---------: | --------- | ------ | --------------- |
| **RSA**     | 2048+ bits | Good      | Slower | Legacy systems  |
| **ED25519** |   256 bits | Excellent | Fast   | **Recommended** |
| **ECDSA**   |   256 bits | Excellent | Fast   | Modern systems  |

### Interview Answer

> I prefer ED25519 keys because they provide strong security with smaller keys and fast cryptographic operations.

---

## Complete Key Setup

```bash
# 1. Generate key
ssh-keygen -t ed25519 -C "your_email@example.com"

# 2. View public key
cat ~/.ssh/id_ed25519.pub

# 3. Copy public key to server
ssh-copy-id user@hostname

# 4. Test connection
ssh user@hostname
```

---

## SSH Agent

The SSH agent stores private keys in memory so you do not need to repeatedly enter the key passphrase.

### Start Agent

```bash
eval $(ssh-agent)
```

### Add Key

```bash
ssh-add ~/.ssh/id_ed25519
```

### List Loaded Keys

```bash
ssh-add -l
```

### Remove Specific Key

```bash
ssh-add -d ~/.ssh/id_ed25519
```

### Remove All Keys

```bash
ssh-add -D
```

### Check Agent

```bash
ps aux | grep ssh-agent
```

---

# 4. Important Files

## Client-Side SSH Files

| File             | Purpose                     | Location  |
| ---------------- | --------------------------- | --------- |
| `id_rsa`         | Private key                 | `~/.ssh/` |
| `id_rsa.pub`     | Public key                  | `~/.ssh/` |
| `id_ed25519`     | ED25519 private key         | `~/.ssh/` |
| `id_ed25519.pub` | ED25519 public key          | `~/.ssh/` |
| `known_hosts`    | Trusted server fingerprints | `~/.ssh/` |
| `config`         | SSH client configuration    | `~/.ssh/` |

Example:

```text
~/.ssh/
├── id_ed25519
├── id_ed25519.pub
├── known_hosts
└── config
```

### Recommended Permissions

```text
Private key       → 600
~/.ssh directory  → 700
authorized_keys   → 600
```

---

## Server-Side SSH Files

| File              | Purpose                             | Location    |
| ----------------- | ----------------------------------- | ----------- |
| `authorized_keys` | Public keys allowed to authenticate | `~/.ssh/`   |
| `sshd_config`     | SSH server configuration            | `/etc/ssh/` |
| `ssh_host_*_key`  | Server host private keys            | `/etc/ssh/` |

Example:

```text
/etc/ssh/
├── sshd_config
├── ssh_host_rsa_key
└── ssh_host_rsa_key.pub

~/.ssh/
└── authorized_keys
```

---

## `authorized_keys`

Contains public keys allowed to authenticate to a user account.

Example:

```text
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIP7... user@machine
```

One key should be stored per line.

### Optional Restrictions

Restrict a key to a specific command:

```text
command="/path/to/script" ssh-ed25519 AAA... key
```

Restrict a key to a specific source IP:

```text
from="192.168.1.*" ssh-ed25519 AAA... key
```

Disable port forwarding for a key:

```text
no-port-forwarding ssh-ed25519 AAA... key
```

---

## `known_hosts`

Stores trusted remote-server host keys.

Example:

```text
hostname.example.com ssh-rsa AAAAB3NzaC1yc2E...
```

During the first connection, SSH asks you to verify the server's fingerprint.

```text
The authenticity of host 'github.com' can't be established.

Are you sure you want to continue connecting?
```

This protects against connecting to an unexpected server.

---

# 5. SSH Config File

## `~/.ssh/config`

The SSH client configuration file simplifies repeated connections.

### Example

```sshconfig
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3
    ForwardAgent yes

Host aws-dev
    HostName ec2-54-123-456-789.compute-1.amazonaws.com
    User ubuntu
    IdentityFile ~/.ssh/aws-key.pem
    Port 22

Host aws-prod
    HostName ec2-54-987-654-321.compute-1.amazonaws.com
    User ubuntu
    IdentityFile ~/.ssh/aws-key.pem
    Port 22

Host github
    HostName github.com
    User git
    IdentityFile ~/.ssh/github-key

Host internal
    HostName internal-server.example.com
    User user
    ProxyCommand ssh -W %h:%p bastion.example.com

Host db
    HostName database.example.com
    User postgres
    Port 2222
    LocalForward 5432 localhost:5432
```

Instead of:

```bash
ssh -i ~/.ssh/aws-key.pem ubuntu@ec2-54-123-456-789.compute-1.amazonaws.com
```

You can simply use:

```bash
ssh aws-dev
```

### Benefits

* Shorter commands
* Centralized connection settings
* Easier management of multiple servers
* Built-in port-forwarding configuration
* Bastion/jump-host support

---

## `sshd_config`

Server-side SSH configuration:

```text
/etc/ssh/sshd_config
```

### Important Settings

```text
Port 22

PermitRootLogin no

PasswordAuthentication yes

PubkeyAuthentication yes

PermitEmptyPasswords no

MaxAuthTries 3

MaxSessions 10

ClientAliveInterval 300

ClientAliveCountMax 3

AllowUsers user1 user2

DenyUsers baduser

AllowGroups devops

LogLevel INFO
```

### Validate Configuration

Always validate before restarting SSH:

```bash
sudo sshd -t
```

### Restart SSH

```bash
sudo systemctl restart sshd
```

On some systems:

```bash
sudo systemctl restart ssh
```

---

# 6. Port Forwarding

## What is SSH Port Forwarding?

SSH can tunnel network traffic through an encrypted SSH connection.

Common uses:

* Access internal services
* Secure database connections
* Access private dashboards
* Avoid exposing internal services publicly
* Tunnel traffic through a bastion host

---

## Local Port Forwarding

### Concept

```text
Local Machine
     │
     │ localhost:8080
     ▼
SSH Server
     │
     │ port 80
     ▼
Target Server
```

### Command

```bash
ssh -L 8080:localhost:80 user@hostname
```

Then:

```text
http://localhost:8080
        ↓
remote-server:80
```

### Forward to Another Internal Server

```bash
ssh -L 8080:internal-server:80 user@hostname
```

This is useful for accessing an internal dashboard without exposing it to the internet.

---

## Remote Port Forwarding

### Concept

```text
Remote Server
     │
     │ remote:8080
     ▼
SSH Tunnel
     │
     ▼
Local Machine
     │
     ▼
localhost:80
```

### Command

```bash
ssh -R 8080:localhost:80 user@hostname
```

Now:

```text
remote-server:8080
        ↓
localhost:80
```

Useful when a remote system needs access to a service running on your local machine.

---

## Dynamic Port Forwarding — SOCKS Proxy

Create a SOCKS proxy:

```bash
ssh -D 1080 user@hostname
```

Configure an application or browser to use:

```text
SOCKS5
localhost:1080
```

Traffic is then routed through the SSH server.

---

## Port Forwarding in SSH Config

```sshconfig
Host app
    HostName app-server.example.com
    User appuser
    LocalForward 8080 localhost:80
    LocalForward 5432 localhost:5432
```

Then:

```bash
ssh app
```

---

# 7. Troubleshooting Scenarios

## Scenario 1 — Connection Refused

### Error

```text
ssh: connect to host example.com port 22: Connection refused
```

### Diagnosis

```bash
# Check SSH service
sudo systemctl status sshd

# Check listening ports
sudo ss -tulpn | grep :22

# Check firewall
sudo iptables -L -n | grep 22
sudo ufw status

# Check connectivity
ping example.com
```

For cloud environments, also check the instance security group and confirm port `22` is allowed.

### Solution

```bash
sudo systemctl start sshd
sudo systemctl enable sshd

sudo ufw allow 22

sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
```

---

## Scenario 2 — Permission Denied (Public Key)

### Error

```text
Permission denied (publickey,password)
```

### Diagnosis

```bash
# Check SSH directory
ls -la ~/.ssh/

# Check authorized_keys
ls -la ~/.ssh/authorized_keys

# Verbose connection
ssh -v user@hostname

# Check loaded keys
ssh-add -l

# Check server authentication logs
sudo tail -f /var/log/auth.log
```

### Expected Permissions

```text
~/.ssh/                  → 700
~/.ssh/authorized_keys   → 600
private key              → 600
```

### Fix Permissions

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chmod 600 ~/.ssh/id_rsa
```

Copy the public key again:

```bash
ssh-copy-id user@hostname
```

Or:

```bash
cat ~/.ssh/id_rsa.pub | ssh user@hostname \
"mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

---

## Scenario 3 — Host Key Verification Failed

### Error

```text
WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!
```

### Possible Causes

* Server was reinstalled
* Host keys were regenerated
* Server identity changed
* Possible man-in-the-middle attack

### Remove Old Host Key

```bash
ssh-keygen -R hostname
```

Or manually edit:

```bash
vim ~/.ssh/known_hosts
```

Then reconnect:

```bash
ssh user@hostname
```

> **Important:** Only remove the old key after verifying that the server is legitimate.

---

## Scenario 4 — SSH Agent Not Found

### Error

```text
Could not open a connection to your authentication agent.
```

### Solution

```bash
eval $(ssh-agent)
ssh-add ~/.ssh/id_rsa
```

---

## Scenario 5 — SSH Timeout

### Error

```text
ssh: connect to host example.com port 22: Connection timed out
```

### Diagnosis

```bash
# Check reachability
ping example.com

# Check firewall
sudo iptables -L -n
sudo ufw status

# Test port
nc -zv example.com 22

# Check network route
traceroute example.com

# Check server load
uptime
top
```

### Possible Solutions

* Check firewall rules
* Check cloud security groups
* Verify network connectivity
* Check server health
* Verify SSH is listening
* Restart SSH if necessary

---

## Scenario 6 — Root Login Denied

### Cause

Root SSH login may be disabled for security.

Check:

```bash
cat /etc/ssh/sshd_config | grep PermitRootLogin
```

Expected secure configuration:

```text
PermitRootLogin no
```

### Recommended Approach

Connect using a normal user:

```bash
ssh ubuntu@hostname
```

Then elevate:

```bash
sudo su -
```

Or:

```bash
sudo -i
```

### Interview Answer

> I avoid direct root SSH access. I connect as a regular user and use sudo for privileged operations because this improves security and auditability.

---

## Scenario 7 — SSH Connection is Slow

### Problem

SSH takes several seconds to establish a connection.

### Diagnosis

```bash
ssh -v user@hostname
```

Common causes mentioned in the guide:

* DNS reverse lookup
* GSSAPI authentication

### Possible Configuration

Server:

```bash
echo "UseDNS no" >> /etc/ssh/sshd_config
sudo systemctl restart sshd
```

Client:

```bash
echo "GSSAPIAuthentication no" >> ~/.ssh/config
echo "GSSAPIDelegateCredentials no" >> ~/.ssh/config
```

---

## Scenario 8 — Too Many Authentication Failures

### Error

```text
Too many authentication failures
```

### Check Logs

```bash
sudo tail -f /var/log/auth.log
```

Check fail2ban:

```bash
sudo systemctl status fail2ban
```

### Prevention

* Use key-based authentication
* Disable password authentication
* Use strong credentials
* Use fail2ban
* Monitor authentication logs

---

## Scenario 9 — Port Forwarding Not Working

### Check Server Configuration

```bash
cat /etc/ssh/sshd_config | grep AllowTcpForwarding
```

Expected:

```text
AllowTcpForwarding yes
```

Check whether the local port is already in use:

```bash
sudo ss -tulpn | grep :8080
```

### Enable Forwarding

```bash
echo "AllowTcpForwarding yes" >> /etc/ssh/sshd_config
sudo systemctl restart sshd
```

---

## Scenario 10 — SCP Permission Denied

### Check Destination

```bash
ssh user@hostname "ls -ld /path/to/destination"
```

Check permissions:

```bash
ssh user@hostname "ls -l /path/to/destination"
```

### Workaround

Copy to a directory the user can write to:

```bash
scp file user@hostname:/tmp/
```

Then move using sudo:

```bash
ssh user@hostname "sudo mv /tmp/file /destination/"
```

---

# 8. Interview Questions & Answers

## Q1. How do you connect to a remote server via SSH?

```bash
ssh user@hostname
ssh -i key.pem user@hostname
ssh -p 2222 user@hostname
```

---

## Q2. How do you copy files using SSH?

For individual files:

```bash
scp localfile user@hostname:/path/
```

For efficient synchronization:

```bash
rsync -avz /local/ user@hostname:/remote/
```

---

## Q3. SSH vs Telnet?

> SSH encrypts communication between the client and server, while Telnet transmits data in plain text. SSH should be preferred for secure remote administration.

---

## Q4. How do you generate SSH keys?

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

---

## Q5. How do you copy your SSH key to a server?

```bash
ssh-copy-id user@hostname
```

Or:

```bash
cat ~/.ssh/id_ed25519.pub | ssh user@hostname \
"cat >> ~/.ssh/authorized_keys"
```

---

## Q6. What is SSH Agent?

> SSH agent stores private keys in memory so the user does not have to repeatedly enter the key passphrase. It is useful when working with multiple SSH connections and automation tools such as Ansible.

---

## Q7. How do you disable root login?

Edit:

```text
/etc/ssh/sshd_config
```

Set:

```text
PermitRootLogin no
```

Then:

```bash
sudo systemctl restart sshd
```

---

## Q8. What is SSH Port Forwarding?

> SSH port forwarding tunnels network traffic through an SSH connection. It is useful for securely accessing internal services without exposing those services directly to the internet.

Example:

```bash
ssh -L 5432:localhost:5432 db-server
```

---

## Q9. Private Key vs Public Key?

> The private key remains secret on the client and should never be shared. The public key is placed on the server in `~/.ssh/authorized_keys`. During authentication, the server uses the public key to verify that the client possesses the corresponding private key.

---

## Q10. What is `~/.ssh/authorized_keys`?

> `authorized_keys` contains the public keys that are permitted to authenticate to a particular user account on the server.

---

## Q11. How do you troubleshoot SSH connection issues?

A practical sequence:

```text
1. ssh -v user@hostname
        ↓
2. Check SSH service
   systemctl status sshd
        ↓
3. Check listening port
   ss -tulpn | grep :22
        ↓
4. Check firewall/security groups
        ↓
5. Check authentication
        ↓
6. Check SSH permissions
        ↓
7. Check authentication logs
```

Useful commands:

```bash
ssh -v user@hostname
sudo systemctl status sshd
sudo ss -tulpn | grep :22
sudo iptables -L -n
sudo tail -f /var/log/auth.log
```

---

## Q12. What is fail2ban?

> fail2ban monitors authentication logs and temporarily blocks IP addresses that repeatedly fail authentication. It helps protect SSH against brute-force attacks.

---

## Q13. How do you change the SSH port?

Edit:

```text
/etc/ssh/sshd_config
```

Set:

```text
Port 2222
```

Restart:

```bash
sudo systemctl restart sshd
```

Connect:

```bash
ssh -p 2222 user@hostname
```

---

## Q14. What is SSH connection multiplexing?

> SSH connection multiplexing allows multiple SSH sessions to reuse an existing SSH connection, reducing connection setup time for repeated connections to the same server.

---

## Q15. What is a Bastion Host / Jump Host?

> A bastion host is a security gateway used as a controlled entry point into private infrastructure. Instead of connecting directly to internal servers, users connect through the bastion host.

```text
Developer
    │
    ▼
Bastion Host
    │
    ├── App Server
    ├── Database
    └── Internal Services
```

---

# 9. Quick Reference Card

## SSH Commands

| Task              | Command                    |
| ----------------- | -------------------------- |
| Basic connection  | `ssh user@host`            |
| Custom port       | `ssh -p 2222 user@host`    |
| Identity file     | `ssh -i key.pem user@host` |
| Execute command   | `ssh user@host "command"`  |
| Verbose debugging | `ssh -v user@host`         |

---

## File Transfer

| Task                      | Command                                 |
| ------------------------- | --------------------------------------- |
| Copy file to remote       | `scp localfile user@host:/path/`        |
| Copy file from remote     | `scp user@host:/path/file ./`           |
| Copy directory            | `scp -r /local/ user@host:/remote/`     |
| Efficient synchronization | `rsync -avz /local/ user@host:/remote/` |

---

## Key Management

| Task               | Command                     |
| ------------------ | --------------------------- |
| Generate key       | `ssh-keygen -t ed25519`     |
| View public key    | `cat ~/.ssh/id_ed25519.pub` |
| Copy key to server | `ssh-copy-id user@host`     |
| Start agent        | `eval $(ssh-agent)`         |
| Add key            | `ssh-add ~/.ssh/id_ed25519` |
| List keys          | `ssh-add -l`                |

---

## Port Forwarding

| Type              | Command                              |
| ----------------- | ------------------------------------ |
| Local forwarding  | `ssh -L 8080:localhost:80 user@host` |
| Remote forwarding | `ssh -R 8080:localhost:80 user@host` |
| SOCKS proxy       | `ssh -D 1080 user@host`              |

---

## SSH Files

| File                     | Purpose                       |
| ------------------------ | ----------------------------- |
| `~/.ssh/id_ed25519`      | Private key                   |
| `~/.ssh/id_ed25519.pub`  | Public key                    |
| `~/.ssh/authorized_keys` | Allowed public keys on server |
| `~/.ssh/known_hosts`     | Trusted server keys           |
| `~/.ssh/config`          | Client configuration          |
| `/etc/ssh/sshd_config`   | Server configuration          |

---

# 10. Production Troubleshooting Checklist

## Cannot Connect

```text
1. Is the server reachable?
   → ping hostname

2. Is SSH running?
   → systemctl status sshd

3. Is SSH listening on the expected port?
   → sudo ss -tulpn | grep :22

4. Is a firewall blocking the connection?
   → sudo iptables -L -n
   → Check cloud security groups

5. Are credentials correct?
   → Check username and key

6. Are SSH permissions correct?
   → ~/.ssh = 700
   → authorized_keys = 600

7. Has the host key changed?
   → ssh-keygen -R hostname
```

---

## Permission Denied

```text
1. Check username

2. Check SSH agent
   → ssh-add -l

3. Check authorized_keys
   → cat ~/.ssh/authorized_keys

4. Check sshd_config

5. Check authentication logs
   → /var/log/auth.log
```

---

# 11. Security Best Practices

## Server-Side

### 1. Disable Root Login

```text
PermitRootLogin no
```

### 2. Prefer Key-Based Authentication

```text
PasswordAuthentication no
```

### 3. Change Default Port

```text
Port 2222
```

### 4. Restrict SSH Users

```text
AllowUsers user1 user2
```

### 5. Use fail2ban

Protect against repeated brute-force attempts.

### 6. Keep the System Updated

```bash
sudo apt update && sudo apt upgrade
```

### 7. Monitor SSH Logs

```bash
sudo tail -f /var/log/auth.log
```

---

## Client-Side

1. Use ED25519 keys.
2. Protect private keys with a passphrase.
3. Use SSH agent where appropriate.
4. Never share private keys.
5. Verify server host keys/fingerprints.
6. Keep private keys at secure permissions.

---

# 12. Common Interview Traps

## Trap 1 — Root Login

❌ **Bad Answer:**

> I connect as root.

✅ **Good Answer:**

> I connect as a regular user and use `sudo` for privileged operations.

---

## Trap 2 — Password Authentication

❌ **Bad Answer:**

> I use password authentication for servers.

✅ **Good Answer:**

> I prefer key-based authentication because it is more secure and suitable for automation.

---

## Trap 3 — Copying SSH Keys

❌ **Bad Answer:**

> I copy the private key to the server.

✅ **Good Answer:**

> I keep the private key on the client and copy only the public key to `~/.ssh/authorized_keys` on the server.

---

## Trap 4 — SSH Agent

❌ **Bad Answer:**

> I type the passphrase every time.

✅ **Good Answer:**

> I use SSH agent to securely manage loaded keys and avoid repeatedly entering the passphrase.

---

# 13. Key Takeaways

1. **SSH is essential for remote server administration.**
2. **Prefer key-based authentication.**
3. **Never directly SSH as root unless there is a specific justified requirement.**
4. **Keep private keys secure with appropriate permissions.**
5. **Use SSH agent to manage keys efficiently.**
6. **Use port forwarding to securely access internal services.**
7. **Use `~/.ssh/config` to simplify repeated connections.**
8. **Use `ssh -v` when troubleshooting connection problems.**
9. **Disable unnecessary authentication methods and root access.**
10. **Know `scp`, `rsync`, and `sftp` for file transfers.**

---

# 14. Commands to Memorize

## Basic SSH

```bash
ssh user@hostname
ssh -i key.pem user@hostname
ssh -p 2222 user@hostname
ssh -v user@hostname
```

## File Transfer

```bash
scp file user@hostname:/path/
rsync -avz /local/ user@hostname:/remote/
```

## Key Management

```bash
ssh-keygen -t ed25519
ssh-copy-id user@hostname
ssh-add ~/.ssh/id_ed25519
```

## Port Forwarding

```bash
ssh -L 8080:localhost:80 user@hostname
ssh -R 8080:localhost:80 user@hostname
ssh -D 1080 user@hostname
```

---

# 🎯 Interview Focus

For a **DevOps / Platform Engineer interview**, you should be able to explain and practically use:

```text
SSH
├── SSH architecture
├── Client vs sshd
├── SSH authentication
├── Public vs private keys
├── ED25519 vs RSA
├── ssh-agent
├── authorized_keys
├── known_hosts
├── ~/.ssh/config
├── /etc/ssh/sshd_config
├── scp
├── rsync
├── sftp
├── Local port forwarding
├── Remote port forwarding
├── SOCKS proxy
├── Bastion / Jump host
├── SSH troubleshooting
├── SSH security
└── fail2ban
```

### Most Important Commands

```bash
# Connect
ssh user@hostname
ssh -i key.pem user@hostname
ssh -p 2222 user@hostname

# Debug
ssh -v user@hostname

# Keys
ssh-keygen -t ed25519
ssh-copy-id user@hostname
ssh-add ~/.ssh/id_ed25519

# File transfer
scp file user@hostname:/path/
rsync -avz /local/ user@hostname:/remote/

# Port forwarding
ssh -L 8080:localhost:80 user@hostname
ssh -R 8080:localhost:80 user@hostname
ssh -D 1080 user@hostname

# Configuration
sudo sshd -t
sudo systemctl restart sshd

# Troubleshooting
sudo systemctl status sshd
sudo ss -tulpn | grep :22
sudo tail -f /var/log/auth.log
ssh-keygen -R hostname
```
