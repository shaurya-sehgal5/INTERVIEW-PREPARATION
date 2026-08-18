# Linux Section 5: CPU, Memory & Load

> **Interview Goal:** Understand CPU usage, RAM, load average, swap, I/O wait, OOM Killer, and how to troubleshoot a slow or overloaded Linux server.

---

# 1. CPU

CPU usage tells you how much processing capacity is being consumed.

### Important

**High CPU does NOT automatically mean something is wrong.**

It could be:

* Expected workload
* Batch processing
* Compilation
* Application bug
* Infinite loop

The important question is:

> **Which process is consuming the CPU and why?**

---

# 2. Memory / RAM

RAM is used by running applications and the operating system.

High memory usage can cause:

```text
High RAM usage
      ↓
Memory pressure
      ↓
Swap usage
      ↓
Performance degradation
      ↓
If memory is exhausted
      ↓
OOM Killer
```

### Important

Linux may use available memory for:

* Applications
* Cache
* Buffers

So seeing high `used` memory does **not automatically mean the server is out of memory**.

For practical troubleshooting, pay attention to:

```bash
free -h
```

especially the **available** memory.

---

# 3. Load Average

Load average represents work waiting for system resources.

For this section, remember:

> **Load includes processes waiting for CPU and processes waiting for I/O.**

`uptime` displays:

```text
1 minute
5 minutes
15 minutes
```

Example:

```text
load average: 0.75, 1.20, 1.50
```

---

# 4. Load Average vs CPU Cores

For a **4-core CPU**:

```text
Load 1 → 25%
Load 2 → 50%
Load 3 → 75%
Load 4 → 100%
Load > 4 → More work than available CPU capacity
```

### Important

A load average above the number of CPU cores indicates that the system has more runnable/I/O-waiting work than the available capacity can handle.

### Interview Answer

> **"Load average represents processes waiting for CPU or I/O. I compare the load average with the number of CPU cores to determine whether the system is overloaded."**

---

# 5. `uptime` — Quick System Health

```bash
uptime
```

Example:

```text
10:30:01 up 5 days, 2:15, 2 users, load average: 0.75, 1.20, 1.50
```

Shows:

* Current time
* System uptime
* Number of users
* 1-minute load
* 5-minute load
* 15-minute load

### Interview Usage

When someone says:

> "The server suddenly became slow."

Start with:

```bash
uptime
```

Then investigate CPU, memory and I/O.

---

# 6. `top` — Real-Time Monitoring

```bash
top
```

Important columns:

| Column    | Meaning        |
| --------- | -------------- |
| `PID`     | Process ID     |
| `USER`    | Process owner  |
| `%CPU`    | CPU usage      |
| `%MEM`    | Memory usage   |
| `TIME+`   | Total CPU time |
| `COMMAND` | Process name   |

### Useful Shortcuts

| Key | Action                    |
| --- | ------------------------- |
| `P` | Sort by CPU               |
| `M` | Sort by memory            |
| `1` | Show individual CPU usage |
| `q` | Quit                      |

---

# 7. `htop`

```bash
htop
```

`htop` provides a more interactive view of processes.

Use it when installed, but know `top` because it is commonly available on Linux servers.

---

# 8. `ps` — Process Snapshot

### All processes

```bash
ps aux
```

### Top CPU processes

```bash
ps aux --sort=-%cpu | head
```

### Top memory processes

```bash
ps aux --sort=-%mem | head
```

### Specific process

```bash
ps -fp 1234
```

---

# 9. `free` — Memory Information

```bash
free -h
```

Example:

```text
               total   used   free   available
Mem:            15G    6.7G   2.3G     8.9G
Swap:            0B      0B     0B       0B
```

### ⭐ Important Interview Point

Don't focus only on:

```text
free
```

Look at:

```text
available
```

because it is a better indication of memory available to applications.

### Interview Answer

> **"`free -h` shows memory and swap usage. For determining whether applications have enough memory available, I pay particular attention to the available value rather than just free memory."**

---

# 10. Swap

Swap is disk space used when RAM is under memory pressure.

Conceptually:

```text
RAM becomes constrained
        ↓
Pages can move to swap
        ↓
Disk is much slower than RAM
        ↓
Performance can degrade
```

Check swap:

```bash
free -h
```

or:

```bash
cat /proc/swaps
```

### Important

Swap usage does not automatically mean the server is broken.

But **heavy/continuous swap activity** can indicate memory pressure.

---

# 11. `vmstat` — System Performance

```bash
vmstat 1 5
```

Means:

> Collect statistics every 1 second, 5 times.

Important columns:

| Column | Meaning                         |
| ------ | ------------------------------- |
| `r`    | Runnable processes / CPU demand |
| `b`    | Blocked processes / I/O wait    |
| `si`   | Swap in                         |
| `so`   | Swap out                        |
| `us`   | User CPU %                      |
| `sy`   | System CPU %                    |
| `id`   | Idle CPU %                      |
| `wa`   | I/O wait %                      |

---

# 12. Understanding `%wa`

`wa` means:

> **I/O wait**

High I/O wait means CPU time is being spent waiting for I/O operations.

Typical investigation:

```bash
top
```

Look at:

```text
%wa
```

Then:

```bash
iostat -x 1 5
```

to investigate disk performance.

### Interview Answer

> **"`%wa` represents I/O wait. High I/O wait can indicate that processes are waiting on disk I/O, so I would investigate disk performance using tools such as `iostat`."**

---

# 13. `dmesg` — Kernel Messages

View recent kernel messages:

```bash
dmesg | tail -20
```

Search for errors:

```bash
dmesg | grep -i error
```

Search for OOM events:

```bash
dmesg | grep -i oom
```

---

# 14. Troubleshooting: CPU at 100%

Suppose:

```text
Server CPU = 100%
```

### Step 1 — Check overall load

```bash
uptime
top
```

### Step 2 — Identify the process

Inside `top`:

```text
P
```

Or:

```bash
ps aux --sort=-%cpu | head -10
```

### Step 3 — Inspect the process

```bash
ps -fp <PID>
```

Find out what command is actually running.

### Step 4 — Check logs

If it is a service:

```bash
journalctl -u SERVICE -n 50
```

Or application logs:

```bash
tail -f /var/log/app.log
```

### Step 5 — Decide what to do

```text
Expected batch job?
        ↓
Let it finish

Application bug?
        ↓
Investigate/restart

Infinite loop?
        ↓
Fix application/restart
```

### ❌ Don't say in an interview:

> "I'll just kill -9 the process."

First determine **why** CPU is high.

---

# 15. Troubleshooting: Low Memory

### Step 1 — Check memory

```bash
free -h
```

### Step 2 — Find biggest memory consumers

```bash
ps aux --sort=-%mem | head -10
```

or:

```bash
top
```

Press:

```text
M
```

### Step 3 — Check swap

```bash
free -h
cat /proc/swaps
```

### Step 4 — Check for OOM events

```bash
dmesg | grep -i oom
```

### Step 5 — Take action

Possible actions:

* Restart offending service
* Increase memory
* Optimize application

---

# 16. Troubleshooting: CPU and Memory Normal but Server is Slow

If:

```text
CPU → Normal
RAM → Normal
```

but the server is still slow, investigate **I/O**.

### Check I/O wait

```bash
top
```

Look at:

```text
%wa
```

### Check disk performance

```bash
iostat -x 1 5
```

### Find which process is doing I/O

```bash
sudo iotop
```

### Check disk space

```bash
df -h
```

---

# 17. Disk Full

Check filesystem usage:

```bash
df -h
```

Find largest directories:

```bash
du -sh /* | sort -h | tail -10
```

Find large files:

```bash
find / -type f -size +100M 2>/dev/null
```

---

# 18. OOM Killer

**OOM = Out Of Memory**

When the system runs out of memory, Linux can invoke the **OOM Killer** to terminate processes so the system can continue operating rather than completely failing.

### Check OOM events

```bash
dmesg | grep -i oom
```

```bash
dmesg | grep -i "killed process"
```

```bash
journalctl -k | grep -i oom
```

### Interview Answer

> **"The OOM Killer is invoked when the system is under severe memory pressure and needs to reclaim memory by terminating a process. I would check kernel logs to determine whether an application was killed due to OOM."**

---

# 19. Production Troubleshooting Flow

```text
Server is slow
      ↓
uptime
      ↓
Check load average
      ↓
top
      ↓
 ┌──────────────┬───────────────┬──────────────┐
 ↓              ↓               ↓
CPU high     Memory high      CPU/RAM normal
 ↓              ↓               ↓
Find PID      free -h          Check I/O
 ↓              ↓               ↓
ps -fp PID    Find process     %wa
                ↓               ↓
              Swap/OOM         iostat
                                ↓
                              Disk
```

---

# 20. Command Summary

| Task                 | Command                       |
| -------------------- | ----------------------------- |
| Overall health       | `uptime`                      |
| Live processes       | `top` / `htop`                |
| Top CPU processes    | `ps aux --sort=-%cpu \| head` |
| Top memory processes | `ps aux --sort=-%mem \| head` |
| Memory               | `free -h`                     |
| Performance          | `vmstat 1`                    |
| Disk I/O             | `iostat -x 1`                 |
| Kernel messages      | `dmesg \| tail`               |
| OOM events           | `dmesg \| grep -i oom`        |

---

# 21. High-Value Interview Questions

### Q1. What does `uptime` show?

> **"It shows system uptime, number of users, and the 1-, 5-, and 15-minute load averages."**

---

### Q2. What is load average?

> **"Load average represents processes waiting for CPU or I/O. I compare it with the number of CPU cores to determine whether the system is overloaded."**

---

### Q3. What is a good load average?

Don't answer with an absolute number.

Answer based on CPU cores.

> **"It depends on the number of CPU cores. On a 4-core system, a sustained load around 4 represents roughly full CPU capacity; significantly above 4 indicates more work than the CPU capacity can handle."**

---

### Q4. `%wa` is high. What does it mean?

> **"`%wa` is I/O wait. Processes are waiting for I/O, so I would investigate disk performance using `iostat` and related tools."**

---

### Q5. How do you find the process using the most memory?

```bash
top
```

Press:

```text
M
```

or:

```bash
ps aux --sort=-%mem | head
```

---

### Q6. What is OOM Killer?

> **"When the system experiences severe memory pressure and cannot satisfy memory allocations, the OOM mechanism can terminate a process to reclaim memory and prevent the system from completely running out of memory."**

---

### Q7. CPU is at 100%. What do you do?

> **"First I check overall load with `uptime` and `top`, identify the process consuming CPU, inspect it with `ps`, check its logs, and determine whether the workload is expected or caused by an application problem. I don't immediately kill the process."**

---

# ⭐ Interview Revision Sheet

```text
CPU
→ Processing capacity

RAM
→ Memory used by applications/system

Load Average
→ CPU + I/O waiting work

uptime
→ System uptime + load average

top
→ Real-time processes

P
→ Sort top by CPU

M
→ Sort top by memory

free -h
→ Memory + swap

available
→ Important memory value

vmstat
→ CPU/memory/system performance

r
→ Runnable processes

b
→ Blocked processes

si/so
→ Swap in/out

us
→ User CPU

sy
→ System CPU

id
→ Idle CPU

wa
→ I/O wait

dmesg
→ Kernel messages

OOM Killer
→ Kills processes under severe memory pressure
```

## 🔥 Must-Know Commands

```bash
uptime

top
htop

ps aux
ps aux --sort=-%cpu | head
ps aux --sort=-%mem | head
ps -fp <PID>

free -h
cat /proc/swaps

vmstat 1 5

dmesg | tail -20
dmesg | grep -i oom

iostat -x 1 5
sudo iotop

df -h
```

> **Section 5 is interview-ready when you can diagnose "server is slow", "CPU is 100%", "RAM is exhausted", and "CPU/RAM are normal but application is slow" without randomly killing processes.**
