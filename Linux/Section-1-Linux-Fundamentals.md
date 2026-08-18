# Linux Section 1: Fundamentals & Filesystem

> **Interview Goal:** Understand Linux fundamentals, filesystem structure, paths, essential commands, and what happens when a command is executed.

---

## 1. Linux = Kernel, Ubuntu = Distribution

| Term             | Meaning                                                            | Example                    |
| ---------------- | ------------------------------------------------------------------ | -------------------------- |
| **Kernel**       | Core of the OS that manages CPU, memory, hardware, processes, etc. | Linux kernel               |
| **Distribution** | Linux kernel + user-space tools + package manager + applications   | Ubuntu, RHEL, Amazon Linux |

### Interview Answer

> **"Linux is the kernel. Ubuntu is a Linux distribution that bundles the Linux kernel with GNU tools, package managers, and applications to provide a complete operating system."**

---

## 2. Kernel Space vs User Space

```text
User Space
  Applications
  nginx, Node.js, Python, Bash
        ↓
    System Calls
        ↓
Kernel Space
  CPU, Memory, Disk, Network
        ↓
     Hardware
```

### Key Point

Applications normally cannot directly control hardware.

They request resources from the kernel through **system calls**.

### Interview Answer

> **"User-space applications interact with hardware through system calls provided by the kernel."**

---

## 3. Shell vs Terminal

| Term         | Meaning                                           | Examples                    |
| ------------ | ------------------------------------------------- | --------------------------- |
| **Terminal** | Interface/window used to interact with the system | GNOME Terminal, SSH session |
| **Shell**    | Program that interprets commands                  | Bash, Zsh, Fish             |

### Command Flow

```text
User
 ↓
Terminal
 ↓
Shell
 ↓
Command
 ↓
Kernel
 ↓
Output
```

### Interview Answer

> **"A terminal is the interface where I type commands. A shell is the program that interprets those commands. Bash is a shell, not an operating system."**

---

# 4. Linux Filesystem Hierarchy

| Directory | Purpose                                       |
| --------- | --------------------------------------------- |
| `/`       | Filesystem root; starting point of everything |
| `/etc`    | System/application configuration              |
| `/var`    | Variable data such as logs and caches         |
| `/home`   | Normal users' home directories                |
| `/root`   | Root user's home directory                    |
| `/tmp`    | Temporary files                               |
| `/usr`    | User-space programs and libraries             |
| `/dev`    | Device files                                  |
| `/proc`   | Virtual process/kernel information            |
| `/sys`    | Virtual device/kernel information             |
| `/run`    | Runtime state since boot                      |
| `/opt`    | Optional/additional software                  |
| `/mnt`    | Temporary mount point                         |

### Important

`/proc` and `/sys` are **virtual filesystems**.

They don't contain normal files stored permanently on disk. The kernel dynamically provides the information when you read them.

### Important Examples

```bash
/proc/cpuinfo
/proc/<PID>/
/sys/block/
/var/log/
/etc/nginx/
/home/user/
```

### Interview Answer

> **"`/proc` is a virtual filesystem that exposes process and kernel information. Its data is generated dynamically by the kernel rather than stored as normal files on disk."**

---

# 5. Linux Paths

| Symbol | Meaning                       |
| ------ | ----------------------------- |
| `.`    | Current directory             |
| `..`   | Parent directory              |
| `~`    | Current user's home directory |
| `/`    | Filesystem root               |

### Absolute Path

Starts from `/`.

```bash
/etc/nginx/nginx.conf
/var/log/app.log
/home/user/project
```

### Relative Path

Starts from the current directory.

```bash
config.yaml
./config.yaml
../config.yaml
```

### Interview Question

**Q: Absolute vs relative path?**

> **"An absolute path starts from the filesystem root `/`, while a relative path starts from the current working directory."**

---

# 6. Essential Linux Commands

## Navigation

```bash
pwd                     # Print current working directory
ls                      # List files
ls -la                  # List all files with details
cd /etc                 # Change directory
cd ~                    # Go to home directory
cd ..                   # Go to parent directory
```

---

## File Operations

```bash
mkdir dir               # Create directory
touch file.txt          # Create empty file
cp source destination   # Copy file
mv source destination   # Move/rename file
rm file                 # Delete file
rm -rf directory        # Delete directory
```

> ⚠️ `rm -rf` is destructive. Use it carefully.

---

## Viewing Files

```bash
cat file                # Print entire file
less file               # Interactive file viewer
head file               # First 10 lines
tail file               # Last 10 lines
tail -f file            # Follow file changes live
```

### DevOps Usage

```bash
tail -f /var/log/app.log
```

Useful for watching application logs in real time.

---

# 7. Finding Files

```bash
find / -name "file.txt"
find /var/log -type f
find /opt -type d
find / -size +100M
```

### Common Interview Pattern

**Find all files larger than 100 MB:**

```bash
find / -type f -size +100M 2>/dev/null
```

---

# 8. File Information

```bash
file filename           # Identify file type
stat filename           # Detailed metadata
which command           # Executable location
whereis command         # Binary/source/man-page locations
```

### Example

```bash
which nginx
stat app.log
file script.sh
```

---

# 9. Environment Variables

```bash
echo $PATH
echo $HOME

export VAR=value

env
printenv
```

### Important

`PATH` tells the shell where to search for executable commands.

Example:

```bash
echo $PATH
```

When you run:

```bash
nginx
```

the shell searches directories listed in `$PATH` for the executable.

---

# 10. Important Linux Files

| File               | Purpose                            |
| ------------------ | ---------------------------------- |
| `/etc/passwd`      | User account information           |
| `/etc/shadow`      | Password hashes; restricted access |
| `/etc/group`       | Group information                  |
| `/etc/sudoers`     | sudo permissions                   |
| `/etc/hosts`       | Local hostname-to-IP mappings      |
| `/etc/resolv.conf` | DNS resolver configuration         |

---

# 11. What Happens When You Type a Command?

```text
You
 ↓
Terminal
 ↓
Shell (Bash)
 ↓
Shell searches PATH
 ↓
Executable is found
 ↓
Process is created
 ↓
Process requests resources from kernel
 ↓
Kernel accesses CPU / memory / disk / network
 ↓
Output is returned
 ↓
Terminal displays output
```

### Interview Answer

> **"When I type a command, the terminal passes it to the shell. The shell finds the executable using PATH, creates or executes the process, and the process interacts with the kernel for required resources."**

---

# 12. High-Value Interview Questions

### Q1. Is Ubuntu Linux?

> **"Ubuntu is a Linux distribution that uses the Linux kernel."**

---

### Q2. Is Bash an operating system?

> **"No. Bash is a shell that interprets commands."**

---

### Q3. Difference between `/` and `/root`?

> **"`/` is the filesystem root. `/root` is the home directory of the root user."**

---

### Q4. What is `/proc`?

> **"`/proc` is a virtual filesystem that exposes process and kernel information. The data is generated dynamically by the kernel."**

---

### Q5. What is the difference between terminal and shell?

> **"A terminal is the interface. A shell is the command interpreter running inside that interface."**

---

### Q6. Absolute vs relative path?

> **"An absolute path starts from `/`; a relative path starts from the current working directory."**

---

# 13. Troubleshooting Flow

### Scenario: Application Cannot Find Configuration

```text
Application cannot find configuration
              ↓
Where should configuration exist?
              ↓
/etc?
Application directory?
Environment variable?
              ↓
Does the file exist?
              ↓
ls / find
              ↓
What does the file contain?
              ↓
cat / less
              ↓
Who owns the file?
              ↓
ls -l / stat
              ↓
Can the application access it?
              ↓
Check permissions
```

---

# 14. Commands to Memorize

```bash
pwd
ls
ls -la
cd
mkdir
touch
cp
mv
rm
find
file
stat
which
whereis
cat
less
head
tail
tail -f
echo
env
printenv
```

## ⭐ Interview Revision — Must Know

```text
Linux       → Kernel
Ubuntu      → Distribution
Bash        → Shell
Terminal    → Interface
/           → Filesystem root
/root       → Root user's home
/etc        → Configuration
/var        → Logs/variable data
/home       → User home
/proc       → Virtual process/kernel information
/sys        → Virtual device/kernel information
PATH        → Executable search locations
```

> **If you can explain these concepts and commands without looking at the README, Section 1 is interview-ready.**
