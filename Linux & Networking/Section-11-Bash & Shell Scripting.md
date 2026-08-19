# Bash & Shell Scripting

> **Interview-Focused Revision Guide**
> **Focus:** Practical Bash commands, scripting fundamentals, automation patterns, and interview-ready answers.

---

## 📚 Table of Contents

1. [Core Concepts](#1-core-concepts)
2. [Essential Scripting Elements](#2-essential-scripting-elements)
3. [Common Automation Scripts](#3-common-automation-scripts)
4. [Interview Questions & Answers](#4-interview-questions--answers)
5. [Quick Reference Card](#5-quick-reference-card)
6. [Commands to Memorize](#6-commands-to-memorize)

---

# 1. Core Concepts

## What is Bash?

**Bash** stands for **Bourne Again Shell**.

It is:

* A shell
* A scripting language
* Commonly used on Linux systems
* Widely used for automation
* Important for system administration
* Frequently used in DevOps workflows

### Interview Answer

> Bash is a shell and scripting language used to automate tasks on Linux. It is essential in DevOps for repetitive tasks, deployment scripts, system administration, and automation.

---

## Shebang (`#!`)

The **shebang** specifies which interpreter should execute the script.

```bash
#!/bin/bash
```

Other examples:

```bash
#!/usr/bin/env bash
#!/bin/sh
#!/usr/bin/python
#!/usr/bin/env node
```

### Common Examples

| Shebang               | Interpreter                |
| --------------------- | -------------------------- |
| `#!/bin/bash`         | Bash                       |
| `#!/usr/bin/env bash` | Bash found through `$PATH` |
| `#!/bin/sh`           | POSIX shell                |
| `#!/usr/bin/python`   | Python                     |
| `#!/usr/bin/env node` | Node.js                    |

### Interview Answer

> The shebang tells the operating system which interpreter should be used to execute a script. For example, `#!/bin/bash` tells the system to execute the script using Bash.

---

## Making a Script Executable

Create a script:

```bash
vim script.sh
```

Add:

```bash
#!/bin/bash

echo "Hello World"
```

Make it executable:

```bash
chmod +x script.sh
```

Run it:

```bash
./script.sh
```

Or directly through Bash:

```bash
bash script.sh
```

### Difference

```bash
./script.sh
```

requires executable permission.

```bash
bash script.sh
```

explicitly invokes Bash and does not require the script itself to have executable permission.

---

# 2. Essential Scripting Elements

## 2.1 Variables

### Define Variables

```bash
NAME="John"
AGE=25
```

### Use Variables

```bash
echo "$NAME is $AGE years old"
```

> No spaces around `=` when assigning variables.

Correct:

```bash
NAME="John"
```

Incorrect:

```bash
NAME = "John"
```

---

## Command Substitution

Use command output as a variable:

```bash
DATE=$(date)
FILES=$(ls -la)
```

Example:

```bash
DISK=$(df -h / | awk 'NR==2 {print $5}')
```

---

## Read User Input

```bash
read -p "Enter name: " USER_NAME

echo "Hello $USER_NAME"
```

---

## Positional Arguments

For a script:

```bash
./script.sh hello world
```

Variables:

```bash
echo "Script: $0"
echo "First argument: $1"
echo "Second argument: $2"
echo "All arguments: $@"
echo "Number of arguments: $#"
```

### Important Variables

| Variable | Meaning             |
| -------- | ------------------- |
| `$0`     | Script name         |
| `$1`     | First argument      |
| `$2`     | Second argument     |
| `$@`     | All arguments       |
| `$#`     | Number of arguments |

---

# 2.2 Conditions

## If / Else

```bash
if [ "$NAME" == "John" ]; then
    echo "Hello John"
elif [ "$NAME" == "Jane" ]; then
    echo "Hello Jane"
else
    echo "Hello Stranger"
fi
```

---

## File Tests

Check whether a file exists:

```bash
if [ -f /etc/nginx/nginx.conf ]; then
    echo "File exists"
fi
```

Check whether a directory exists:

```bash
if [ -d /var/log ]; then
    echo "Directory exists"
fi
```

---

## String Tests

Check whether a variable is empty:

```bash
if [ -z "$VAR" ]; then
    echo "VAR is empty"
fi
```

Check whether it is not empty:

```bash
if [ -n "$VAR" ]; then
    echo "VAR is not empty"
fi
```

---

## Numeric Comparisons

```bash
if [ "$AGE" -gt 18 ]; then
    echo "Adult"
fi
```

### Operators

| Operator | Meaning               |
| -------- | --------------------- |
| `-eq`    | Equal                 |
| `-ne`    | Not equal             |
| `-lt`    | Less than             |
| `-le`    | Less than or equal    |
| `-gt`    | Greater than          |
| `-ge`    | Greater than or equal |

---

## Common File Tests

| Test | Meaning               |
| ---- | --------------------- |
| `-f` | Regular file exists   |
| `-d` | Directory exists      |
| `-e` | File/directory exists |
| `-r` | Readable              |
| `-w` | Writable              |
| `-x` | Executable            |
| `-s` | File is not empty     |

---

# 2.3 Loops

## For Loop — Files

```bash
for file in *.log; do
    echo "Processing $file"
    gzip "$file"
done
```

---

## For Loop — Range

```bash
for i in {1..10}; do
    echo "Number: $i"
done
```

---

## For Loop — List

```bash
for service in nginx mysql redis; do
    systemctl status "$service"
done
```

---

## While Loop — Read File

```bash
while read line; do
    echo "Line: $line"
done < file.txt
```

---

## Infinite While Loop

```bash
while true; do
    echo "Running..."
    sleep 5
done
```

---

## Until Loop

```bash
until ping -c 1 google.com; do
    echo "Waiting for network..."
    sleep 2
done
```

---

# 2.4 Functions

Functions allow reusable pieces of Bash logic.

## Define a Function

```bash
function check_service() {
    local service=$1

    systemctl is-active "$service" >/dev/null 2>&1

    if [ $? -eq 0 ]; then
        echo "$service is running"
    else
        echo "$service is not running"
    fi
}
```

Call it:

```bash
check_service nginx
check_service mysql
```

---

## Function Returning a Value

```bash
function get_disk_usage() {
    df -h / | awk 'NR==2 {print $5}' | sed 's/%//'
}
```

Use it:

```bash
USAGE=$(get_disk_usage)

if [ "$USAGE" -gt 80 ]; then
    echo "Disk usage is high: $USAGE%"
fi
```

---

## `local`

Use `local` when a variable should remain local to a function:

```bash
function example() {
    local NAME="John"
}
```

---

# 2.5 Exit Codes

Every command returns an exit status.

```bash
command

if [ $? -eq 0 ]; then
    echo "Success"
else
    echo "Failed with code: $?"
fi
```

### Important Exit Codes

|  Code | Meaning                 |
| ----: | ----------------------- |
|   `0` | Success                 |
|   `1` | General error           |
|   `2` | Misuse of shell builtin |
| `126` | Command cannot execute  |
| `127` | Command not found       |

---

## Script Exit

```bash
exit 0
```

Successful execution.

```bash
exit 1
```

Generic failure.

```bash
exit 2
```

Misuse of a shell builtin.

---

# 2.6 Command Substitution

### Old Syntax

```bash
DATE=`date`
```

### Preferred Syntax

```bash
DATE=$(date)
```

Use `$(...)` for modern Bash scripts.

Examples:

```bash
DATE=$(date)

FILES=$(ls -la)

DISK=$(df -h / | awk 'NR==2 {print $5}')
```

Example inside a condition:

```bash
if [ "$(du -s /var/log | awk '{print $1}')" -gt 1000000 ]; then
    echo "Logs are too large"
fi
```

---

# 3. Common Automation Scripts

## 3.1 Service Health Check

```bash
#!/bin/bash

# monitor.sh - Check critical services

SERVICES=("nginx" "mysql" "redis")

for service in "${SERVICES[@]}"; do
    if systemctl is-active "$service" >/dev/null 2>&1; then
        echo "✓ $service is running"
    else
        echo "✗ $service is not running"
        echo "Starting $service..."
        systemctl start "$service"
    fi
done
```

### What It Does

1. Defines critical services.
2. Loops through each service.
3. Checks whether the service is active.
4. Starts the service if it is down.

---

# 3.2 Disk Space Alert

```bash
#!/bin/bash

# disk_alert.sh - Alert if disk usage > 80%

THRESHOLD=80

USAGE=$(df -h / | awk 'NR==2 {print $5}' | sed 's/%//')

if [ "$USAGE" -gt "$THRESHOLD" ]; then
    echo "WARNING: Disk usage is $USAGE%"
    echo "Top 10 largest directories:"
    du -sh /* | sort -h | tail -10
fi
```

### Purpose

Detects when root filesystem usage exceeds a configured threshold.

---

# 3.3 Log Rotation Script

```bash
#!/bin/bash

# rotate_logs.sh - Compress and rotate logs

LOG_DIR="/var/log/app"
DAYS=7

find "$LOG_DIR" -name "*.log" -type f -mtime +"$DAYS" |
while read file; do
    echo "Compressing: $file"
    gzip "$file"
done

# Delete logs older than 30 days
find "$LOG_DIR" -name "*.log.gz" -type f -mtime +30 -delete
```

### Purpose

* Compress logs older than 7 days.
* Delete compressed logs older than 30 days.

---

# 3.4 Backup Script

```bash
#!/bin/bash

# backup.sh - Backup important directories

BACKUP_DIR="/backup"
DATE=$(date +%Y%m%d)

DIRS_TO_BACKUP=(
    "/var/www"
    "/etc"
    "/var/log"
)

for dir in "${DIRS_TO_BACKUP[@]}"; do
    NAME=$(basename "$dir")

    tar -czf "$BACKUP_DIR/${NAME}_$DATE.tar.gz" "$dir"

    echo "Backed up: $dir"
done

# Delete backups older than 7 days
find "$BACKUP_DIR" -name "*.tar.gz" -type f -mtime +7 -delete
```

### Purpose

Backs up important directories and removes old backups according to the retention period.

---

# 3.5 Deployment Script

```bash
#!/bin/bash

# deploy.sh - Simple deployment script

APP_DIR="/opt/myapp"
BACKUP_DIR="/backup/app"
DATE=$(date +%Y%m%d_%H%M%S)

# Backup current version
echo "Creating backup..."
tar -czf "$BACKUP_DIR/app_$DATE.tar.gz" "$APP_DIR"

# Stop service
echo "Stopping service..."
systemctl stop myapp

# Deploy new code
echo "Deploying new code..."
cp -r dist/* "$APP_DIR/"

# Fix permissions
chown -R appuser:appgroup "$APP_DIR"

# Start service
echo "Starting service..."
systemctl start myapp

# Health check
sleep 5

if curl -s http://localhost:8080/health | grep -q "OK"; then
    echo "Deployment successful!"
else
    echo "Deployment failed! Rolling back..."
    # Rollback logic here
fi
```

### Deployment Flow

```text
Backup Current Version
        ↓
Stop Service
        ↓
Deploy New Code
        ↓
Fix Permissions
        ↓
Start Service
        ↓
Health Check
        ↓
 ┌──────┴──────┐
 ↓             ↓
Success      Failure
               ↓
           Rollback
```

---

# 3.6 Parse Logs Script

```bash
#!/bin/bash

# parse_errors.sh - Extract and count errors from logs

LOG_FILE="/var/log/app/app.log"
DATE=$(date +%Y-%m-%d)

echo "=== Error Report for $DATE ==="

grep -i "error" "$LOG_FILE" |
    grep "$DATE" |
    cut -d' ' -f4- |
    sort |
    uniq -c |
    sort -nr |
    head -10

echo ""

echo "Total errors: $(grep -c "error" "$LOG_FILE")"
```

### Purpose

* Extract errors.
* Filter by date.
* Group similar errors.
* Sort by frequency.
* Display the top 10.

---

# 3.7 Find and Kill Script

```bash
#!/bin/bash

# kill_process.sh - Find and kill process by name

if [ -z "$1" ]; then
    echo "Usage: $0 <process_name>"
    exit 1
fi

PID=$(pgrep -f "$1" | head -1)

if [ -z "$PID" ]; then
    echo "Process $1 not found"
    exit 1
fi

echo "Found $1 with PID: $PID"

read -p "Kill process? (y/n): " CONFIRM

if [ "$CONFIRM" == "y" ]; then
    kill "$PID"
    echo "Process killed"
fi
```

### Flow

```text
Check Argument
      ↓
Find PID
      ↓
PID Found?
  ┌───┴───┐
 No       Yes
 ↓         ↓
Exit     Ask Confirmation
           ↓
        Kill Process
```

---

# 4. Interview Questions & Answers

## Q1. What is the shebang in a Bash script?

> The shebang, such as `#!/bin/bash`, tells the operating system which interpreter should execute the script.

---

## Q2. How do you make a Bash script executable?

```bash
chmod +x script.sh
```

Then:

```bash
./script.sh
```

---

## Q3. Difference Between `$@` and `$*`?

> `$@` treats each positional argument as a separate word, while `$*` treats all positional arguments as a single string. `$@` is preferred when you need to preserve individual arguments, especially when arguments contain spaces.

---

## Q4. How do you check if a file exists?

```bash
if [ -f /path/to/file ]; then
    echo "File exists"
fi
```

---

## Q5. Difference Between `$?` and `$$`?

| Variable | Meaning                         |
| -------- | ------------------------------- |
| `$?`     | Exit code of the last command   |
| `$$`     | PID of the current shell/script |

Example:

```bash
echo $?
```

Check the previous command's exit status.

```bash
echo $$
```

Display the current shell's PID.

---

## Q6. How do you handle errors in Bash?

Using conditional execution:

```bash
command || {
    echo "Command failed"
    exit 1
}
```

Or:

```bash
set -e
```

Use an error trap:

```bash
trap 'echo "Error on line $LINENO"' ERR
```

### Interview Answer

> I use exit codes, conditional execution, `set -e` where appropriate, and traps to detect and handle failures.

---

## Q7. How do you read a file line by line?

```bash
while read line; do
    echo "$line"
done < file.txt
```

---

## Q8. How do you check if a command exists?

```bash
command -v nginx >/dev/null 2>&1

if [ $? -eq 0 ]; then
    echo "nginx installed"
fi
```

---

## Q9. Difference Between `=` and `==`?

> In Bash, `=` can be used for string comparison inside `[ ]`. `==` is commonly used for string comparison inside `[[ ]]`, which is Bash-specific.

Example:

```bash
if [ "$A" = "$B" ]; then
    echo "Equal"
fi
```

Bash-specific:

```bash
if [[ "$A" == "$B" ]]; then
    echo "Equal"
fi
```

---

## Q10. How do you create a function in Bash?

```bash
function my_function() {
    echo "Hello"
}
```

Or:

```bash
my_function() {
    echo "Hello"
}
```

Call it:

```bash
my_function
```

---

## Q11. How Do You Use Environment Variables?

Use an existing variable:

```bash
echo "$HOME"
```

Export a variable:

```bash
export MY_VAR="value"
```

Load variables from `.env`:

```bash
source .env
```

Then:

```bash
echo "$DATABASE_URL"
```

---

## Q12. How Do You Iterate Over Files?

```bash
for file in /path/*.log; do
    echo "$file"
done
```

---

## Q13. How Do You Check Whether a Directory Exists and Create It if Not?

```bash
if [ ! -d /path/to/dir ]; then
    mkdir -p /path/to/dir
fi
```

---

## Q14. Difference Between Single and Double Quotes?

### Double Quotes

Allow variable expansion:

```bash
NAME="John"

echo "$NAME"
```

Output:

```text
John
```

### Single Quotes

Treat content literally:

```bash
echo '$NAME'
```

Output:

```text
$NAME
```

### Interview Answer

> Double quotes allow variable and command expansion, while single quotes treat the content literally.

---

## Q15. How Do You Run a Command Only If the Previous Command Succeeded?

Use `&&`:

```bash
command1 && command2
```

`command2` runs only if `command1` succeeds.

Use `||`:

```bash
command1 || command2
```

`command2` runs only if `command1` fails.

---

# 5. Quick Reference Card

## Variables

| Task           | Syntax                 |
| -------------- | ---------------------- |
| Define         | `NAME="value"`         |
| Use            | `echo "$NAME"`         |
| Command output | `DATE=$(date)`         |
| Arguments      | `$1`, `$2`, `$@`, `$#` |

---

## Conditions

| Task             | Syntax                             |   |                 |
| ---------------- | ---------------------------------- | - | --------------- |
| If/else          | `if [ condition ]; then ... fi`    |   |                 |
| File exists      | `[ -f file ]`                      |   |                 |
| Directory exists | `[ -d dir ]`                       |   |                 |
| String equal     | `[ "$A" == "$B" ]`                 |   |                 |
| Numeric equal    | `[ "$A" -eq "$B" ]`                |   |                 |
| Empty string     | `[ -z "$VAR" ]`                    |   |                 |
| AND              | `[ condition1 ] && [ condition2 ]` |   |                 |
| OR               | `[ condition1 ]                    |   | [ condition2 ]` |

---

## Loops

| Task      | Syntax                                |
| --------- | ------------------------------------- |
| Files     | `for file in *.log; do ... done`      |
| Range     | `for i in {1..10}; do ... done`       |
| Read file | `while read line; do ... done < file` |
| Infinite  | `while true; do ... done`             |

---

## Functions

| Task           | Syntax                    |
| -------------- | ------------------------- |
| Define         | `function name() { ... }` |
| Call           | `name arg1 arg2`          |
| Local variable | `local VAR="value"`       |
| Return         | `return 0`                |

---

## Common Patterns

| Task             | Syntax                         |
| ---------------- | ------------------------------ |
| Check success    | `if [ $? -eq 0 ]; then ... fi` |
| Exit on error    | `set -e`                       |
| Debug mode       | `set -x`                       |
| Error trap       | `trap 'echo Error' ERR`        |
| Sleep            | `sleep 5`                      |
| Load environment | `source .env`                  |
| Export variable  | `export VAR=value`             |
| Read input       | `read -p "..." VAR`            |

---

# 6. Commands to Memorize

## Bash Basics

```bash
#!/bin/bash

chmod +x script.sh

./script.sh

bash script.sh
```

---

## Variables & Arguments

```bash
NAME="value"

echo "$NAME"

echo "$1"
echo "$@"
echo "$#"
echo "$?"
echo "$$"
```

---

## Conditions

```bash
if [ condition ]; then
    ...
elif [ condition ]; then
    ...
else
    ...
fi
```

---

## Loops

```bash
for file in *.log; do
    ...
done
```

```bash
while read line; do
    ...
done < file
```

```bash
while true; do
    ...
done
```

---

## Functions

```bash
function name() {
    ...
}
```

or:

```bash
name() {
    ...
}
```

---

## Error Handling

```bash
set -e
```

```bash
trap 'echo Error' ERR
```

```bash
command1 && command2
```

```bash
command1 || command2
```

---

## Environment

```bash
source .env
```

```bash
export VAR="value"
```

---

## Input & Timing

```bash
read -p "Enter value: " VAR
```

```bash
sleep 5
```

---

# 🎯 Interview Focus

For a **DevOps / Platform Engineer interview**, you should be comfortable with:

```text
Bash & Shell Scripting
├── Bash fundamentals
├── Shebang
├── Script execution
├── Variables
├── Environment variables
├── Positional arguments
├── Conditions
├── File tests
├── String comparisons
├── Numeric comparisons
├── for loops
├── while loops
├── until loops
├── Functions
├── local variables
├── Exit codes
├── Command substitution
├── Error handling
├── set -e
├── trap
├── Source / export
├── Log automation
├── Backup automation
├── Deployment scripts
├── Health checks
└── Process automation
```

### 🔥 Must-Know Syntax

```bash
#!/bin/bash

# Variables
NAME="value"

# Arguments
$0
$1
$@
$#

# Exit status / PID
$?
$$

# Condition
if [ condition ]; then
    ...
fi

# File tests
[ -f file ]
[ -d directory ]

# Loops
for file in *.log; do
    ...
done

while read line; do
    ...
done < file

# Functions
function name() {
    ...
}

# Command substitution
RESULT=$(command)

# Error handling
set -e
trap 'echo Error' ERR

# Environment
source .env
export VAR="value"

# Command chaining
command1 && command2
command1 || command2
```

> **Interview priority:** Be able to write a small Bash script from scratch that checks a service, validates disk usage, parses logs, handles failures, accepts arguments, and performs a basic deployment or backup workflow.
