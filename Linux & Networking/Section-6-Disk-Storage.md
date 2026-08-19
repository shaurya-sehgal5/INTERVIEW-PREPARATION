# Linux Section 6: Disk, Storage & Filesystems

> **Interview Goal:** Understand disk space, inodes, `df`, `du`, filesystem usage, disk I/O, mounting, and how to troubleshoot "No space left on device".

---

# 1. Disk Space vs Inodes

This is one of the **most important concepts** in Linux storage troubleshooting.

| Concept        | Tracks                      |
| -------------- | --------------------------- |
| **Disk space** | Amount of data stored       |
| **Inodes**     | Number of files/directories |

### Important

You can have:

```text
20 GB disk space free
```

but still be unable to create files because:

```text
Inodes = 100% used
```

### Memory Trick

```text
Disk space → How much DATA?

Inodes → How many FILES?
```

---

# 2. `df` — Filesystem Usage

`df` shows filesystem-level disk usage.

### Human-readable

```bash
df -h
```

### Specific filesystem

```bash
df -h /
```

### Inode usage

```bash
df -i
```

---

# 3. Understanding `df -h`

Example:

```text
Filesystem      Size  Used  Avail  Use%  Mounted on
/dev/vda1        50G   30G   18G   63%   /
```

Important columns:

| Column       | Meaning           |
| ------------ | ----------------- |
| `Filesystem` | Device/filesystem |
| `Size`       | Total size        |
| `Used`       | Used space        |
| `Avail`      | Available space   |
| `Use%`       | Percentage used   |
| `Mounted on` | Mount point       |

---

# 4. `du` — Directory Usage

`du` shows how much space directories/files are consuming.

### Current directory

```bash
du -sh *
```

### Specific directory

```bash
du -sh /var
```

### Sort directories by size

```bash
du -sh /* | sort -h
```

### Top 10 largest directories

```bash
du -sh /* | sort -h | tail -10
```

---

# 5. `df` vs `du`

### ⭐ Very Common Interview Question

| `df`                         | `du`                                    |
| ---------------------------- | --------------------------------------- |
| Filesystem-level             | Directory/file-level                    |
| Shows partition usage        | Shows directory usage                   |
| Good for overall disk health | Good for finding what's consuming space |

### Interview Answer

> **"`df` shows filesystem-level disk usage, while `du` shows how much space individual directories and files are consuming. I use `df` for the overall filesystem and `du` to find where the space is being used."**

---

# 6. Find Large Files

### Files larger than 100 MB

```bash
find / -type f -size +100M -exec ls -lh {} \; 2>/dev/null
```

### Files larger than 1 GB

```bash
find / -type f -size +1G -exec ls -lh {} \; 2>/dev/null
```

### Largest log files

```bash
find /var/log -type f -exec ls -lh {} \; | sort -k5 -h | tail -10
```

---

# 7. Check Disk I/O

If the disk is slow:

```bash
iostat -x 1 5
```

Look at:

* Utilization
* I/O rates
* Latency-related metrics
* Device performance

Also check:

```bash
top
```

for:

```text
%wa
```

### Process-level I/O

```bash
sudo iotop
```

This helps identify which process is performing heavy disk I/O.

---

# 8. Check Inodes

```bash
df -i
```

Specific filesystem:

```bash
df -i /
```

Example:

```text
Filesystem     Inodes     IUsed     IFree     IUse%
/dev/vda1      3276800   3276800   0         100%
```

Here:

```text
Disk space may still be available
BUT
No new files can be created
```

because all inodes are consumed.

---

# 9. Why Can Inodes Become Full?

A system can consume huge numbers of inodes by creating enormous numbers of small files.

Example:

```text
Application
   ↓
Creates millions of tiny files
   ↓
Inodes exhausted
   ↓
"No space left on device"
```

Even if:

```text
df -h
```

shows free GBs.

---

# 10. Production Scenario: Disk is Full

Suppose:

```text
Disk usage = 95%
```

### Step 1 — Find full filesystem

```bash
df -h
```

### Step 2 — Find largest directories

```bash
du -sh /* | sort -h | tail -10
```

### Common Culprits

```text
/var/log
/var/lib/docker
/tmp
/var/tmp
```

---

# 11. Cleaning Journal Logs

Check journal usage:

```bash
journalctl --disk-usage
```

Reduce journal size by size:

```bash
sudo journalctl --vacuum-size=500M
```

Or remove logs older than 7 days:

```bash
sudo journalctl --vacuum-time=7d
```

---

# 12. Docker Disk Usage

Docker can consume significant disk space.

The source specifically highlights:

```text
/var/lib/docker
```

as a common disk-space consumer.

Basic cleanup:

```bash
docker system prune -f
```

Remove unused images:

```bash
docker image prune -f
```

> ⚠️ Understand what Docker resources are unused before running cleanup commands on a production host.

---

# 13. Old Log Files

Find old logs:

```bash
find /var/log -name "*.log" -mtime +30
```

The source provides:

```bash
find /var/log -name "*.log" -mtime +30 -delete
```

> ⚠️ Deleting logs is a production-impacting action. Understand retention requirements before deleting them.

---

# 14. Production Scenario: Disk is Slow

### Step 1 — Check I/O wait

```bash
top
```

Look at:

```text
%wa
```

### Step 2 — Check disk statistics

```bash
iostat -x 1 5
```

### Step 3 — Identify process doing I/O

```bash
sudo iotop
```

### Step 4 — Determine whether workload is expected

Examples:

```text
Backup?
Database migration?
Log processing?
Unexpected application behavior?
```

### Possible Actions

* Identify high-I/O process
* Determine whether workload is expected
* Optimize application
* Move to faster storage if necessary

---

# 15. Production Scenario: "No Space Left" But `df -h` Shows Free Space

This is a classic interview scenario.

Suppose:

```text
Application:
No space left on device
```

But:

```bash
df -h
```

shows:

```text
20G available
```

### Next Step

Check inodes:

```bash
df -i
```

If:

```text
IUse% = 100%
```

then you've exhausted inodes.

### Fix

Find areas containing huge numbers of small files and remove unnecessary ones.

---

# 16. Finding Too Many Files

The source suggests checking file counts with commands such as:

```bash
find / -type d -exec ls -1 {} \; 2>/dev/null | wc -l
```

and:

```bash
ls -la /var/spool | wc -l
```

The goal is to identify directories containing unusually large numbers of files.

---

# 17. Mounting Disks

### Show mounted filesystems

```bash
mount
```

### List block devices

```bash
lsblk
```

### List partitions

```bash
sudo fdisk -l
```

---

# 18. Mount a Disk

Example:

```bash
sudo mount /dev/sdb1 /mnt
```

This mounts:

```text
/dev/sdb1
```

at:

```text
/mnt
```

### Unmount

```bash
sudo umount /mnt
```

---

# 19. Understand the Mount Concept

Think of it as:

```text
Disk / Partition
      ↓
/dev/sdb1
      ↓
mount
      ↓
/mnt
```

After mounting:

```text
/mnt
```

provides access to the filesystem on `/dev/sdb1`.

---

# 20. Check Filesystem Health

### Filesystem check

```bash
sudo fsck /dev/sdb1
```

> ⚠️ The source notes that the filesystem should be **unmounted** when performing `fsck`.

### Disk health

If `smartmontools` is installed:

```bash
sudo smartctl -a /dev/sda
```

---

# 21. Disk Troubleshooting Flow

```text
Disk problem
     ↓
df -h
     ↓
Is filesystem full?
     ↓
YES
 ↓
du -sh /* | sort -h
 ↓
Find large directory
 ↓
Find large files
 ↓
Logs? Docker? Temporary files?
```

If `df -h` looks normal:

```text
        ↓
Check df -i
        ↓
Inodes full?
        ↓
YES
        ↓
Find directories with huge numbers of files
```

If disk isn't full:

```text
        ↓
Is server slow?
        ↓
Check %wa
        ↓
iostat
        ↓
iotop
```

---

# 22. High-Value Interview Questions

### Q1. `df` vs `du`?

> **"`df` shows filesystem-level usage. `du` shows directory/file-level usage. I use `df` to identify a full filesystem and `du` to find what is consuming the space."**

---

### Q2. Disk is 95% full. What do you do?

> **"First I run `df -h` to identify the full filesystem. Then I use `du -sh` to find the largest directories, and investigate common consumers such as logs, Docker data, and temporary files."**

Commands:

```bash
df -h
du -sh /* | sort -h | tail -10
```

---

### Q3. Application says "No space left on device" but 20 GB is free. What do you check?

> **"I would check inode usage with `df -i`. The filesystem can have free disk space while all inodes are exhausted."**

```bash
df -i
```

---

### Q4. What are inodes?

> **"Inodes are filesystem structures used to track files and directories. A system can run out of inodes even when disk space is still available."**

---

### Q5. How do you find files larger than 1 GB?

```bash
find / -type f -size +1G -exec ls -lh {} \; 2>/dev/null
```

---

### Q6. `%wa` is high. What does it mean?

> **"`%wa` indicates I/O wait. High I/O wait can mean processes are waiting on disk I/O, so I would investigate with `iostat` and `iotop`."**

---

### Q7. How do you check disk I/O?

```bash
iostat -x 1 5
```

and:

```bash
sudo iotop
```

---

### Q8. How do you find mounted disks?

```bash
lsblk
mount
```

---

### Q9. How do you mount a disk?

```bash
sudo mount /dev/sdb1 /mnt
```

Unmount:

```bash
sudo umount /mnt
```

---

# ⭐ Interview Revision Sheet

```text
Disk Space
→ Amount of DATA stored

Inodes
→ Number of FILES/directories

df
→ Filesystem-level usage

du
→ Directory/file-level usage

df -h
→ Human-readable disk usage

df -i
→ Inode usage

du -sh *
→ Directory sizes

find -size
→ Find large files

iostat
→ Disk I/O statistics

iotop
→ Process-level I/O

%wa
→ I/O wait

lsblk
→ Block devices

mount
→ Mounted filesystems

umount
→ Unmount filesystem

fsck
→ Filesystem check

smartctl
→ Disk health
```

---

# 🔥 Must-Know Commands

```bash
df -h
df -i

du -sh *
du -sh /var
du -sh /* | sort -h | tail -10

find / -type f -size +100M 2>/dev/null

find / -type f -size +1G -exec ls -lh {} \; 2>/dev/null

iostat -x 1 5
sudo iotop

lsblk
mount
sudo fdisk -l

sudo mount /dev/sdb1 /mnt
sudo umount /mnt

sudo fsck /dev/sdb1
sudo smartctl -a /dev/sda
```

---

# 🚨 Most Important Interview Scenario

### Interviewer:

> **"Your production server says `No space left on device`, but `df -h` shows 20 GB free. What do you do?"**

### Strong Answer:

> **"First I'd check inode usage with `df -i`. Disk space and inode availability are different resources. If the inode usage is 100%, the filesystem can have free GBs but still be unable to create new files. I'd then identify directories containing huge numbers of small files and clean up only unnecessary files according to the application's retention requirements."**

---

# ⭐ Final Mental Model

```text
                 LINUX STORAGE
                       │
        ┌──────────────┼──────────────┐
        ↓              ↓              ↓
    Disk Space       Inodes         I/O
        │              │              │
      df -h          df -i         iostat
        │              │              │
      du -sh        file count     iotop
        │
   Find large files
        │
   Logs / Docker /
   Temporary data
```

> **Section 6 is interview-ready when you can clearly explain `df vs du`, disk space vs inodes, diagnose "No space left on device", find large files/directories, investigate high I/O wait, and explain basic disk mounting.**
