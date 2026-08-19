# Linux Section 3: Processes & Process Management

> **Interview Goal:** Understand programs vs processes, PIDs, PPIDs, process states, threads, signals, background jobs, `/proc`, file descriptors, and process troubleshooting.

---

# 1. Program vs Process

| Term        | Meaning                                |
| ----------- | -------------------------------------- |
| **Program** | Passive executable file stored on disk |
| **Process** | Running instance of a program          |

Example:

```text
/usr/bin/python3
      ↓
   Program
      ↓
python3 app.py
      ↓
   Process
```

A process has its own:

* PID
* Memory/address space
* CPU execution state
* File descriptors
* Environment variables
* Credentials
* Working directory
* PPID

### Interview Answer

> **"A program is a passive set of instructions stored on disk, while a process is an executing instance of that program with its own PID, memory, execution state, and associated resources."**

---

# 2. PID — Process ID

Every running process has a unique **Process ID (PID)**.

```bash
ps
```

Example:

```text
PID   TTY      TIME     CMD
2451  pts/0    00:00:00 bash
2817  pts/0    00:00:00 nginx
```

You can inspect a specific process:

```bash
ps -fp 2817
```

---

# 3. PPID — Parent Process ID

A process can create child processes.

```bash
ps -ef
```

Example:

```text
UID   PID   PPID   CMD
user  3000  2500   python app.py
```

```text
PID 3000  → Child process
PPID 2500 → Parent process
```

### Process Relationship

```text
Parent Process
      │
      ├── Child Process
      │
      └── Child Process
```

---

# 4. PID 1

PID 1 is the **first userspace process started by the kernel**.

Check it:

```bash
ps -p 1
```

On modern Linux systems it is typically:

```text
systemd
```

### Why PID 1 Matters

PID 1:

* Performs system initialization
* Manages/reaps processes
* Becomes the parent of orphaned processes
* Is especially important inside containers

### DevOps Interview Point

In containers, the process running as **PID 1** matters for:

* Signal handling
* Process lifecycle
* Child process reaping

### Interview Answer

> **"PID 1 is the first userspace process. On a typical Linux system it's systemd. In containers, what runs as PID 1 is important because of signal handling and process management."**

---

# 5. Process States

| State | Meaning               |
| ----- | --------------------- |
| `R`   | Running/Runnable      |
| `S`   | Interruptible sleep   |
| `D`   | Uninterruptible sleep |
| `T`   | Stopped               |
| `Z`   | Zombie                |

---

## R — Running/Runnable

Process is:

* Currently executing, or
* Ready to run on the CPU.

---

## S — Interruptible Sleep

Process is sleeping while waiting for something such as:

* I/O
* Event
* Timer

This is very common.

---

## D — Uninterruptible Sleep

Usually means the process is waiting for kernel-level I/O.

```text
Process
   ↓
Waiting for kernel I/O
   ↓
D state
```

### Important

A process stuck in `D` state may not respond to:

```bash
kill -9 PID
```

because the process is stuck waiting inside the kernel.

### Interview Question

**Q: Why can `kill -9` fail?**

> **"If the process is stuck in uninterruptible sleep, or D state, SIGKILL cannot wake it because it's waiting on kernel-level I/O. I would investigate the underlying I/O problem."**

---

# 6. Zombie Process

A zombie is a process that has:

1. Finished execution
2. But whose parent hasn't collected its exit status yet

```text
Child process exits
       ↓
Zombie entry remains
       ↓
Parent calls wait()
       ↓
Zombie entry removed
```

### Important

A zombie:

* Has already finished execution
* Does not consume CPU
* Mainly occupies a process-table entry
* Exists until its parent collects the exit status

### Interview Answer

> **"A zombie is a process that has finished execution but whose parent hasn't collected its exit status. It remains as an entry in the process table."**

---

# 7. Zombie vs Orphan

| Feature       | Zombie                                   | Orphan            |
| ------------- | ---------------------------------------- | ----------------- |
| Process state | Finished                                 | Still running     |
| Parent        | Still exists but hasn't collected status | Parent terminated |
| Re-parented?  | No                                       | Yes               |
| New parent    | —                                        | PID 1             |

### Easy Memory Trick

```text
Zombie → Child is DEAD, parent hasn't collected it

Orphan → Child is ALIVE, parent is DEAD
```

---

# 8. Process vs Thread

A process can contain multiple threads:

```text
Process
│
├── Thread 1
├── Thread 2
└── Thread 3
```

| Feature       | Process      | Thread                |
| ------------- | ------------ | --------------------- |
| Address space | Separate     | Shared                |
| Resources     | Own          | Shared within process |
| Creation      | Heavier      | Lighter               |
| Communication | IPC required | Shared memory         |

### Key Concept

Processes provide isolation.

Threads share the process's memory/resources.

### Interview Answer

> **"A process provides an isolated execution environment with its own address space, while threads are execution units inside a process that share the process's address space and resources."**

---

# 9. `ps` — Process Snapshot

### Current session

```bash
ps
```

### All processes

```bash
ps aux
```

### All processes with parent-child information

```bash
ps -ef
```

### Top CPU-consuming processes

```bash
ps aux --sort=-%cpu | head
```

### Top memory-consuming processes

```bash
ps aux --sort=-%mem | head
```

### Specific PID

```bash
ps -fp 1234
```

---

# 10. `top` — Live Process Monitoring

```bash
top
```

Useful shortcuts:

| Key | Action                    |
| --- | ------------------------- |
| `P` | Sort by CPU               |
| `M` | Sort by memory            |
| `1` | Show individual CPU cores |
| `q` | Quit                      |

### Interview Usage

If CPU suddenly becomes high:

```bash
top
```

Press:

```text
P
```

to find the process consuming the most CPU.

---

# 11. `htop`

```bash
htop
```

`htop` provides a more interactive process-monitoring interface.

---

# 12. Find Processes

### Find PID by process name

```bash
pgrep nginx
```

### Show PID + command

```bash
pgrep -a nginx
```

### Find PID by program

```bash
pidof nginx
```

---

# 13. `pstree`

View the parent-child process hierarchy:

```bash
pstree
```

With PIDs:

```bash
pstree -p
```

Example:

```text
systemd
 ├── sshd
 │    └── bash
 │         └── python
 └── nginx
```

Very useful for understanding which process spawned another process.

---

# 14. Background Jobs

Run a process in the background:

```bash
python app.py &
```

Check jobs:

```bash
jobs
```

Bring job to foreground:

```bash
fg
```

Send job to background:

```bash
bg
```

---

# 15. `nohup`

`nohup` allows a process to continue after the terminal/session disconnects.

```bash
nohup ./app.sh &
```

### Important

For production workloads, prefer:

```text
systemd
Docker
Kubernetes
```

instead of relying on `nohup`.

---

# 16. Linux Signals

Signals are notifications sent to processes.

| Signal    | Number | Meaning              |
| --------- | -----: | -------------------- |
| `SIGTERM` |     15 | Graceful termination |
| `SIGKILL` |      9 | Forceful termination |
| `SIGINT`  |      2 | Interrupt            |
| `SIGHUP`  |      1 | Hangup/reload        |
| `SIGSTOP` |     19 | Stop                 |
| `SIGCONT` |     18 | Continue             |

---

# 17. `SIGTERM` vs `SIGKILL`

## SIGTERM

```bash
kill PID
```

Equivalent to:

```bash
kill -15 PID
```

Requests graceful termination.

The application gets an opportunity to:

* Clean up
* Close files
* Finish work
* Exit gracefully

---

## SIGKILL

```bash
kill -9 PID
```

Equivalent to:

```bash
kill -SIGKILL PID
```

Forcefully terminates the process.

The application cannot catch or handle SIGKILL.

### Correct Production Approach

```text
SIGTERM
   ↓
Wait
   ↓
Still running?
   ↓
SIGKILL
```

### Interview Answer

> **"SIGTERM requests graceful termination, while SIGKILL immediately forces termination. I use SIGTERM first and only use SIGKILL when the process doesn't terminate gracefully."**

---

# 18. Other Useful Signals

### Interrupt

```bash
kill -INT PID
```

Equivalent to:

```text
Ctrl+C
```

### Reload/HUP

```bash
kill -HUP PID
```

### Stop

```bash
kill -STOP PID
```

### Continue

```bash
kill -CONT PID
```

---

# 19. `/proc/<PID>`

Linux exposes process information through:

```text
/proc/<PID>/
```

Example:

```bash
ls /proc/1234
```

### Process Status

```bash
cat /proc/1234/status
```

### Command Line

```bash
cat /proc/1234/cmdline
```

### Environment

```bash
cat /proc/1234/environ
```

### File Descriptors

```bash
ls /proc/1234/fd/
```

---

# 20. File Descriptors

Every process has standard file descriptors.

|  FD | Name   |
| --: | ------ |
| `0` | stdin  |
| `1` | stdout |
| `2` | stderr |

### Redirect stdout

```bash
command > output.log
```

### Redirect stderr

```bash
command 2> error.log
```

### Remember

```text
0 → stdin
1 → stdout
2 → stderr
```

---

# 21. Find Which Process Uses a Port

Using `ss`:

```bash
ss -tulpn | grep :8080
```

Using `lsof`:

```bash
lsof -i :8080
```

### Common DevOps Scenario

Application fails:

```text
EADDRINUSE: address already in use
```

Check:

```bash
sudo ss -tulpn | grep :8080
```

or:

```bash
sudo lsof -i :8080
```

Then identify the PID and investigate the process.

---

# 22. Troubleshooting: High CPU

### Step 1 — Check overall system

```bash
uptime
top
```

### Step 2 — Find high CPU process

```bash
top
```

Press:

```text
P
```

or:

```bash
ps aux --sort=-%cpu | head
```

### Step 3 — Inspect process

```bash
ps -fp <PID>
```

### Step 4 — Check logs

```bash
journalctl -u SERVICE -n 50
```

### Step 5 — Take appropriate action

```text
Expected workload?
    ↓
Let it finish

Application bug?
    ↓
Investigate/restart

Process stuck?
    ↓
Try graceful termination
```

> **Don't automatically `kill -9` a high-CPU process. First determine why it is consuming CPU.**

---

# 23. Troubleshooting: High Memory

```bash
top
```

Press:

```text
M
```

Or:

```bash
ps aux --sort=-%mem | head
```

Then inspect:

```bash
ps -fp <PID>
```

---

# 24. Troubleshooting: Process Won't Die

First:

```bash
kill PID
```

Wait and check:

```bash
ps -fp PID
```

If necessary:

```bash
kill -9 PID
```

If even SIGKILL doesn't work:

```text
Check process state
       ↓
ps / top
       ↓
D state?
       ↓
Investigate underlying I/O issue
```

---

# 25. High-Value Interview Questions

### Q1. Program vs process?

> **"A program is a passive executable file on disk. A process is a running instance of that program with a PID, memory, CPU state, and resources."**

---

### Q2. What is a PID?

> **"PID is the unique process ID assigned to a running process."**

---

### Q3. What is PPID?

> **"PPID is the process ID of the parent process that created or spawned the current process."**

---

### Q4. What is a zombie?

> **"A zombie is a terminated process whose parent hasn't collected its exit status. It remains as a process-table entry."**

---

### Q5. Zombie vs orphan?

> **"A zombie has finished execution but its parent hasn't collected its status. An orphan is still running but its parent has terminated, so it gets re-parented to PID 1."**

---

### Q6. SIGTERM vs SIGKILL?

> **"SIGTERM requests graceful termination. SIGKILL forces immediate termination and cannot be handled by the process. I use SIGTERM first."**

---

### Q7. Why can `kill -9` fail?

> **"A process stuck in D state is waiting for uninterruptible kernel-level I/O, so SIGKILL cannot wake it. The underlying I/O problem must be investigated."**

---

### Q8. What is PID 1?

> **"PID 1 is the first userspace process, typically systemd. It performs system initialization and process management. In containers, the PID 1 process is especially important for signal handling and child-process management."**

---

### Q9. Process vs thread?

> **"Processes have separate address spaces and provide isolation. Threads run inside a process and share its address space and resources."**

---

# ⭐ Interview Revision Sheet

```text
Program       → Passive executable on disk
Process       → Running instance
PID           → Process ID
PPID          → Parent Process ID
PID 1         → First userspace process

R             → Running/Runnable
S             → Interruptible sleep
D             → Uninterruptible sleep
T             → Stopped
Z             → Zombie

Zombie        → Finished, parent hasn't collected status
Orphan        → Running, parent terminated

SIGTERM 15    → Graceful termination
SIGKILL 9     → Forceful termination
SIGINT 2      → Interrupt
SIGHUP 1      → Hangup/reload
SIGSTOP 19    → Stop
SIGCONT 18    → Continue

FD 0          → stdin
FD 1          → stdout
FD 2          → stderr

ps aux        → All processes
ps -ef        → Processes + parent information
top           → Live monitoring
pgrep         → Find process
pstree        → Process hierarchy
kill          → Send signal
ss            → Network sockets
lsof          → Open files/connections
/proc/PID     → Process information
```

## 🔥 Must-Know Commands

```bash
ps aux
ps -ef
ps -fp <PID>

top
ps aux --sort=-%cpu | head
ps aux --sort=-%mem | head

pgrep -a nginx
pidof nginx
pstree -p

kill PID
kill -9 PID

ss -tulpn
lsof -i :8080

ls /proc/<PID>
cat /proc/<PID>/status
cat /proc/<PID>/cmdline
ls /proc/<PID>/fd/
```

> **Section 3 is interview-ready when you can explain PID/PPID, process states, zombie vs orphan, process vs thread, SIGTERM vs SIGKILL, PID 1, and troubleshoot a high-CPU or stuck process using commands rather than guessing.**
