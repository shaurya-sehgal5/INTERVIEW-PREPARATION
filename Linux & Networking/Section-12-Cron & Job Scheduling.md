# Cron & Job Scheduling

> **Interview-Focused Revision Guide**
> **Focus:** Practical job scheduling, cron syntax, automation patterns, troubleshooting, and interview-ready answers.

---

## 📚 Table of Contents

1. [Core Concepts](#1-core-concepts)
2. [Essential Commands](#2-essential-commands)
3. [Important Files](#3-important-files)
4. [Common Scheduling Patterns](#4-common-scheduling-patterns)
5. [Troubleshooting Scenarios](#5-troubleshooting-scenarios)
6. [Interview Questions & Answers](#6-interview-questions--answers)
7. [Quick Reference Card](#7-quick-reference-card)
8. [Commands to Memorize](#8-commands-to-memorize)
9. [Common Interview Traps](#9-common-interview-traps)
10. [Key Takeaways](#10-key-takeaways)

---

# 1. Core Concepts

## What is Cron?

**Cron** is a time-based job scheduler on Linux.

```text
Cron
  ↓
Schedule
  ↓
Execute Commands at Specific Times
```

### Why Cron Matters

Cron is commonly used to automate:

* Repetitive tasks
* Scheduled backups
* Log rotation
* System maintenance
* Monitoring scripts

### Interview Answer

> Cron is a time-based job scheduler that runs commands at specified intervals. It's used for automating recurring tasks like backups, log rotation, and system maintenance.

---

# Cron vs Systemd Timers

| Feature         | Cron          | Systemd Timers           |
| --------------- | ------------- | ------------------------ |
| Standard        | Traditional   | Modern                   |
| Syntax          | Simple        | More complex             |
| Logging         | Basic         | Integrated with journald |
| Dependencies    | No            | Yes                      |
| Email reporting | Yes, via mail | Configurable             |
| Security        | Basic         | More secure              |

### Interview Answer

> Cron is the traditional scheduler. Systemd timers are newer and offer better integration with systemd, dependency management, and logging.

---

# Crontab Syntax

A standard cron entry contains five scheduling fields followed by the command:

```text
* * * * * command-to-execute
│ │ │ │ │
│ │ │ │ └── Day of week (0-7, Sunday=0 or 7)
│ │ │ └──── Month (1-12)
│ │ └────── Day of month (1-31)
│ └──────── Hour (0-23)
└────────── Minute (0-59)
```

### Field Order

```text
minute hour day-of-month month day-of-week command
```

---

## Common Cron Expressions

| Schedule              | Cron Expression |
| --------------------- | --------------- |
| Every minute          | `* * * * *`     |
| Every 5 minutes       | `*/5 * * * *`   |
| Every hour            | `0 * * * *`     |
| Every day at 2 AM     | `0 2 * * *`     |
| Every Sunday at 3 AM  | `0 3 * * 0`     |
| 1st of every month    | `0 0 1 * *`     |
| Every weekday at 9 AM | `0 9 * * 1-5`   |

---

# Special Cron Strings

| String     | Meaning                           |
| ---------- | --------------------------------- |
| `@reboot`  | Run at boot                       |
| `@hourly`  | Run every hour                    |
| `@daily`   | Run every day                     |
| `@weekly`  | Run every Sunday                  |
| `@monthly` | Run on the first day of the month |
| `@yearly`  | Run on the first day of January   |

---

# 2. Essential Commands

## 2.1 Crontab Management

### Edit Current User's Crontab

```bash
crontab -e
```

### View Current User's Jobs

```bash
crontab -l
```

### Remove All Current User's Cron Jobs

```bash
crontab -r
```

> Be careful with `crontab -r` because it removes all cron jobs for the current user.

### Edit Another User's Crontab

```bash
sudo crontab -u username -e
```

### View Another User's Crontab

```bash
sudo crontab -u username -l
```

---

# 2.2 System-Wide Cron Jobs

View the main system crontab:

```bash
cat /etc/crontab
```

View additional cron jobs:

```bash
ls /etc/cron.d/
```

Directory-based jobs:

```bash
ls /etc/cron.daily/
ls /etc/cron.hourly/
ls /etc/cron.weekly/
ls /etc/cron.monthly/
```

---

# 2.3 Cron Logging

On systems using traditional syslog:

```bash
tail -f /var/log/syslog | grep CRON
```

Search existing entries:

```bash
grep CRON /var/log/syslog
```

Using journald:

```bash
journalctl -u cron
```

Follow cron logs:

```bash
journalctl -u cron -f
```

---

# 3. Important Files

## 3.1 `/etc/crontab` — System-Wide Cron

Example:

```text
# /etc/crontab

SHELL=/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# Run backup at 2 AM daily
0 2 * * * root /usr/local/bin/backup.sh
```

### Important Difference

System crontab includes a **user field**:

```text
minute hour day month day-of-week user command
```

---

# 3.2 `/etc/cron.d/` — Additional System Jobs

Example:

```text
# /etc/cron.d/myapp

0 2 * * * root /opt/myapp/cleanup.sh

*/5 * * * * root /opt/myapp/health-check.sh
```

These files are useful for managing application-specific system jobs.

---

# 3.3 `/etc/cron.daily/`, `/etc/cron.hourly/`, etc.

Linux provides directories for periodic jobs:

```text
/etc/cron.hourly/
/etc/cron.daily/
/etc/cron.weekly/
/etc/cron.monthly/
```

Place executable scripts in these directories.

Example:

```text
/etc/cron.daily/
└── backup.sh
```

The exact execution time is controlled by the system's cron/anacron configuration.

---

# 3.4 User-Specific Cron Jobs

Manage personal cron jobs with:

```bash
crontab -e
```

View them:

```bash
crontab -l
```

---

# 4. Common Scheduling Patterns

## 4.1 Backup Schedule

### Daily Backup at 2 AM

```cron
0 2 * * * /opt/scripts/backup.sh
```

### Weekly Full Backup — Sunday at 3 AM

```cron
0 3 * * 0 /opt/scripts/full-backup.sh
```

### Monthly Archive — First Day at 4 AM

```cron
0 4 1 * * /opt/scripts/archive.sh
```

---

# 4.2 Log Rotation

### Rotate Logs Daily at Midnight

```cron
0 0 * * * /usr/sbin/logrotate /etc/logrotate.conf
```

### Archive Old Logs Weekly

```cron
0 1 * * 0 find /var/log -name "*.log" -mtime +7 -gzip
```

> The source contains the `-gzip` example above; in a real Linux environment, verify the command before deploying it because `find` does not generally provide `-gzip` as a standard action.

---

# 4.3 System Maintenance

### Clear Temporary Files Daily at 3 AM

```cron
0 3 * * * find /tmp -type f -atime +1 -delete
```

### Update Package Lists Daily at 4 AM

```cron
0 4 * * * apt update
```

### Check Disk Space at 6 AM

```cron
0 6 * * * /opt/scripts/disk-usage.sh
```

---

# 4.4 Monitoring Scripts

### Health Check Every 5 Minutes

```cron
*/5 * * * * /opt/scripts/health-check.sh
```

### Check Service Status Every Hour

```cron
0 * * * * systemctl status nginx
```

### Check Disk Usage at 8 AM

```cron
0 8 * * * /opt/scripts/disk-check.sh
```

---

# 4.5 Database Maintenance

### Daily PostgreSQL Backup at 1 AM

```cron
0 1 * * * pg_dump dbname > /backup/db_$(date +%Y%m%d).sql
```

### Weekly Vacuum — Sunday at 2 AM

```cron
0 2 * * 0 psql -d dbname -c "VACUUM ANALYZE;"
```

### Delete Backups Older Than 30 Days

```cron
0 3 * * * find /backup -name "*.sql" -mtime +30 -delete
```

---

# 5. Troubleshooting Scenarios

# Scenario 1 — Cron Job Not Running

### Problem

A job has been scheduled but is not executing.

### Step 1 — Check Cron Service

```bash
sudo systemctl status cron
```

### Step 2 — Check Cron Logs

```bash
sudo journalctl -u cron -f
```

Or:

```bash
sudo tail -f /var/log/syslog | grep CRON
```

### Step 3 — Check Crontab

```bash
crontab -l
```

### Step 4 — Check Script Permissions

```bash
ls -l /path/to/script
```

### Step 5 — Make Script Executable

```bash
chmod +x /path/to/script
```

### Step 6 — Run Script Manually

```bash
/path/to/script
```

### Troubleshooting Flow

```text
Cron Job Not Running
        ↓
Check cron service
        ↓
Check cron logs
        ↓
Check crontab syntax
        ↓
Check permissions
        ↓
Check executable bit
        ↓
Run script manually
```

---

# Scenario 2 — Job Runs but Doesn't Work

### Problem

The cron job executes but fails silently.

### Redirect Output

```cron
0 2 * * * /path/to/script.sh > /tmp/cron.log 2>&1
```

Check output:

```bash
cat /tmp/cron.log
```

### Check Cron Environment

```cron
0 2 * * * /usr/bin/env > /tmp/env_cron.log
```

### Use Full Paths

```cron
0 2 * * * /bin/bash /path/to/script.sh
```

---

# Scenario 3 — Environment Variables Missing

### Problem

The script works manually but cannot find commands when executed by cron.

Cron often runs with a limited environment.

## Option 1 — Set PATH in Script

```bash
#!/bin/bash

export PATH=/usr/local/bin:/usr/bin:/bin
```

## Option 2 — Set PATH in Crontab

```cron
PATH=/usr/local/bin:/usr/bin:/bin

0 2 * * * /path/to/script.sh
```

## Option 3 — Use Full Paths

```cron
0 2 * * * /usr/bin/python3 /path/to/script.py
```

### Interview Answer

> Cron runs with a limited environment, so I explicitly set PATH and other required variables or use absolute paths for commands.

---

# Scenario 4 — Job Runs Multiple Times

### Problem

The same task appears to execute multiple times.

Possible cause:

* Duplicate crontab entries
* `/etc/crontab` entry
* `/etc/cron.d/` entry
* `/etc/cron.daily/` script
* Another scheduler

### Check User Crontab

```bash
crontab -l
```

### Check System Crontab

```bash
sudo cat /etc/crontab
```

### Check `/etc/cron.d/`

```bash
ls /etc/cron.d/
```

### Check Periodic Directories

```bash
ls /etc/cron.daily/
```

---

# Scenario 5 — Job Running as Wrong User

### Edit Specific User's Crontab

```bash
sudo crontab -u username -e
```

### System Crontab

System-wide entries specify the user:

```cron
0 2 * * * username /path/to/script.sh
```

---

# Scenario 6 — Email Notifications Not Working

### Check `MAILTO`

```cron
MAILTO=user@example.com
```

### Check Mail Program

```bash
which mail
```

### Check Postfix

```bash
sudo systemctl status postfix
```

### Alternative — Log to System Journal

```cron
0 2 * * * /path/to/script.sh 2>&1 | logger -t myjob
```

Then inspect:

```bash
journalctl -t myjob
```

---

# 6. Interview Questions & Answers

## Q1. What is Cron and What Is It Used For?

> Cron is a time-based job scheduler on Linux. It is used to automate recurring tasks such as backups, log rotation, system maintenance, and monitoring scripts.

---

## Q2. What Does `* * * * * command` Mean?

> It means the command runs every minute of every hour, every day, every month, and every day of the week.

---

## Q3. How Do You Schedule a Job Every 5 Minutes?

```cron
*/5 * * * * /path/to/script.sh
```

---

## Q4. How Do You Schedule a Job at 2 AM Every Sunday?

```cron
0 2 * * 0 /path/to/script.sh
```

---

## Q5. How Do You View All Cron Jobs for the Current User?

```bash
crontab -l
```

---

## Q6. How Do You Edit Cron Jobs?

```bash
crontab -e
```

---

## Q7. How Do You Remove All Cron Jobs?

```bash
crontab -r
```

> **Warning:** This removes all cron jobs for the current user.

---

## Q8. Where Are Cron Logs Stored?

> On systems using syslog, cron logs can be found in `/var/log/syslog`. They can also be viewed with `journalctl -u cron`.

```bash
journalctl -u cron
```

---

## Q9. Difference Between `crontab -e` and `/etc/crontab`?

> `crontab -e` edits user-specific cron jobs. `/etc/crontab` is system-wide and includes an additional field specifying which user should run the command.

### User Crontab

```text
minute hour day month day-of-week command
```

### System Crontab

```text
minute hour day month day-of-week user command
```

---

## Q10. Why Might a Cron Job Not Run?

Common reasons:

1. Cron service is not running.
2. Incorrect cron syntax.
3. Script lacks execute permission.
4. Wrong PATH.
5. Missing environment variables.
6. Incorrect file paths.
7. Script works manually but depends on interactive environment.
8. Wrong user.
9. Duplicate or conflicting scheduling configuration.

---

## Q11. How Do You Run a Job at Boot?

```cron
@reboot /path/to/script.sh
```

---

## Q12. How Do You Handle Environment Variables in Cron?

> Set PATH and required environment variables explicitly in the crontab or script. Use absolute paths for commands and source an environment file from the script when appropriate.

Example:

```bash
#!/bin/bash

export PATH=/usr/local/bin:/usr/bin:/bin

source /path/to/.env
```

---

## Q13. Difference Between Cron and Systemd Timers?

> Cron is a traditional and simple scheduler. Systemd timers are more modern and integrate with systemd, journald, dependencies, and service management.

---

## Q14. How Do You Redirect Cron Output to a File?

```cron
0 2 * * * /path/to/script.sh > /tmp/output.log 2>&1
```

### Meaning

```text
>       → Redirect stdout
2>&1    → Redirect stderr to stdout
```

Therefore, both standard output and errors are written to the same file.

---

## Q15. How Do You Set Up Automatic Log Cleanup Using Cron?

```cron
0 3 * * * find /var/log -name "*.log" -mtime +30 -delete
```

This runs every day at 3 AM and deletes matching `.log` files older than 30 days.

> In production, ensure this aligns with your retention, compliance, and backup policies before using deletion commands.

---

# 7. Quick Reference Card

## Crontab Syntax

| Field        | Values               |
| ------------ | -------------------- |
| Minute       | `0-59`               |
| Hour         | `0-23`               |
| Day of month | `1-31`               |
| Month        | `1-12`               |
| Day of week  | `0-7` (`0` = Sunday) |

### Format

```text
minute hour day-of-month month day-of-week command
```

---

## Common Schedules

| Schedule        | Expression    |
| --------------- | ------------- |
| Every minute    | `* * * * *`   |
| Every 5 minutes | `*/5 * * * *` |
| Every hour      | `0 * * * *`   |
| Daily at 2 AM   | `0 2 * * *`   |
| Weekly Sunday   | `0 2 * * 0`   |
| Monthly 1st     | `0 0 1 * *`   |

---

## Special Strings

| String     | Meaning        |
| ---------- | -------------- |
| `@reboot`  | At system boot |
| `@daily`   | Once daily     |
| `@weekly`  | Once weekly    |
| `@monthly` | Once monthly   |

---

## Essential Commands

| Task                | Command                                |
| ------------------- | -------------------------------------- |
| Edit jobs           | `crontab -e`                           |
| List jobs           | `crontab -l`                           |
| Remove jobs         | `crontab -r`                           |
| View cron logs      | `tail -f /var/log/syslog \| grep CRON` |
| Check service       | `systemctl status cron`                |
| View journal logs   | `journalctl -u cron`                   |
| Follow journal logs | `journalctl -u cron -f`                |

---

# 8. Commands to Memorize

```bash
# Edit cron jobs
crontab -e

# List cron jobs
crontab -l

# Remove all cron jobs
crontab -r

# Edit another user's cron
sudo crontab -u username -e

# View another user's cron
sudo crontab -u username -l

# Check cron service
sudo systemctl status cron

# Follow cron logs
journalctl -u cron -f

# Traditional cron logs
tail -f /var/log/syslog | grep CRON
```

### Must-Know Cron Expressions

```cron
# Every minute
* * * * * command

# Every 5 minutes
*/5 * * * * command

# Every hour
0 * * * * command

# Every day at 2 AM
0 2 * * * command

# Every Sunday at 2 AM
0 2 * * 0 command

# Every weekday at 9 AM
0 9 * * 1-5 command

# At system boot
@reboot command
```

---

# 9. Common Interview Traps

## Trap 1 — Forgetting PATH

❌ **Bad Answer:**

> My script works manually but not in cron.

✅ **Good Answer:**

> Cron has a limited environment and PATH. I use absolute paths for commands and explicitly configure PATH when necessary.

---

## Trap 2 — Not Checking Permissions

❌ **Bad Answer:**

> The script exists, so cron should execute it.

✅ **Good Answer:**

> I check whether the script has the required permissions and execution bit using `ls -l` and `chmod +x`.

---

## Trap 3 — Ignoring Cron Logs

❌ **Bad Answer:**

> I don't know whether the cron job ran.

✅ **Good Answer:**

> I check `/var/log/syslog` for CRON entries or use `journalctl -u cron`. I also redirect stdout and stderr to a log file when troubleshooting.

---

## Trap 4 — Confusing User and System Crontabs

❌ **Bad Answer:**

> `/etc/crontab` works exactly like `crontab -e`.

✅ **Good Answer:**

> User crontabs don't specify the user because they already belong to a user. `/etc/crontab` is system-wide and contains an additional user field.

---

# 10. Key Takeaways

1. **Cron automates recurring tasks.**
2. **Cron syntax is: `minute hour day month day-of-week command`.**
3. **Use `*/5` for every 5 minutes.**
4. **Use `@reboot` to execute a job at boot.**
5. **Use absolute paths in cron jobs.**
6. **Set PATH and required environment variables explicitly.**
7. **Redirect stdout and stderr when debugging.**
8. **Check cron logs using `/var/log/syslog` or `journalctl -u cron`.**
9. **The cron service must be running.**
10. **Scripts must have the required permissions.**
11. **Know the difference between user crontabs and `/etc/crontab`.**
12. **Understand Cron vs Systemd timers.**

---

# 🎯 Interview Focus

For a **DevOps / Platform Engineer interview**, you should be able to confidently explain and use:

```text
Cron & Job Scheduling
├── Cron fundamentals
├── Crontab syntax
├── Five scheduling fields
├── Special strings
├── User crontabs
├── /etc/crontab
├── /etc/cron.d/
├── /etc/cron.daily/
├── /etc/cron.hourly/
├── Cron service
├── Cron logs
├── Environment variables
├── PATH issues
├── Permissions
├── Output redirection
├── Scheduled backups
├── Log cleanup
├── Monitoring jobs
├── Database maintenance
├── @reboot
└── Cron vs Systemd timers
```

### 🔥 Must-Know

```bash
crontab -e
crontab -l
crontab -r

systemctl status cron

journalctl -u cron
journalctl -u cron -f

tail -f /var/log/syslog | grep CRON
```

### 🔥 Must-Know Expressions

```cron
* * * * * command
*/5 * * * * command
0 * * * * command
0 2 * * * command
0 2 * * 0 command
0 9 * * 1-5 command
@reboot command
```

### Interview Mental Model

```text
Cron Job
   │
   ├── Is cron running?
   │
   ├── Is syntax correct?
   │
   ├── Is the script executable?
   │
   ├── Is PATH correct?
   │
   ├── Are environment variables available?
   │
   ├── Is the correct user executing it?
   │
   ├── Are absolute paths used?
   │
   └── Are logs/output being captured?
```

> **Core interview skill:** If a cron job works manually but fails under cron, immediately think **environment, PATH, permissions, user context, absolute paths, and logs**.
