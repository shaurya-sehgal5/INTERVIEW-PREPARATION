# Linux Section 2: Files, Permissions & Ownership

> **Interview Goal:** Understand Linux file types, permissions, ownership, users/groups, symbolic links, ACLs, and how to troubleshoot `Permission denied`.

---

# 1. Linux Treats Almost Everything as a File

Common Linux file types:

| Symbol | Type             | Example                   |
| ------ | ---------------- | ------------------------- |
| `-`    | Regular file     | Config, text, binary      |
| `d`    | Directory        | Folder                    |
| `l`    | Symbolic link    | Shortcut/reference        |
| `c`    | Character device | Keyboard, serial device   |
| `b`    | Block device     | Disk                      |
| `s`    | Socket           | IPC/network communication |
| `p`    | Named pipe       | IPC                       |

---

# 2. Understanding `ls -l`

Example:

```bash
-rwxr-x--- 1 shaurya devops 2048 Aug 17 deploy.sh
```

Breakdown:

```text
-rwxr-x--- 1 shaurya devops 2048 Aug 17 deploy.sh
│├──┤├──┤├──┤
│ │    │    │
│ │    │    └── Others permissions
│ │    └─────── Group permissions
│ └──────────── Owner permissions
└────────────── File type
```

| Part        | Meaning            |
| ----------- | ------------------ |
| `-`         | Regular file       |
| `rwx`       | Owner permissions  |
| `r-x`       | Group permissions  |
| `---`       | Others permissions |
| `1`         | Hard-link count    |
| `shaurya`   | Owner              |
| `devops`    | Group              |
| `2048`      | Size               |
| `Aug 17`    | Modification time  |
| `deploy.sh` | Filename           |

### Interview Pattern

Given:

```bash
-rwxr-x---
```

Think:

```text
Owner  → rwx
Group  → r-x
Others → ---
```

---

# 3. Read, Write, Execute

## Files

| Permission | Meaning            |
| ---------- | ------------------ |
| `r`        | Read file contents |
| `w`        | Modify file        |
| `x`        | Execute file       |

## Directories

Directory permissions are different.

| Permission | Directory Meaning            |
| ---------- | ---------------------------- |
| `r`        | List directory contents      |
| `w`        | Create/delete/rename entries |
| `x`        | Enter/traverse directory     |

### Critical Interview Point

A user can have:

```text
r--
```

on a directory but still be unable to:

```bash
cd directory
```

because `x` permission is required to enter/traverse it.

### Interview Answer

> **"For a directory, read allows listing contents, write allows creating/deleting entries, and execute allows entering or traversing the directory."**

---

# 4. Numeric Permissions

Linux permission values:

| Value | Permission |
| ----: | ---------- |
|   `4` | Read       |
|   `2` | Write      |
|   `1` | Execute    |

Add them together:

| Number | Permission    |
| -----: | ------------- |
|    `7` | `rwx` = 4+2+1 |
|    `6` | `rw-` = 4+2   |
|    `5` | `r-x` = 4+1   |
|    `4` | `r--`         |
|    `3` | `-wx` = 2+1   |
|    `2` | `-w-`         |
|    `1` | `--x`         |
|    `0` | `---`         |

---

# 5. Common Permission Values

| Permission | Symbolic    | Meaning                         | Typical Use             |
| ---------: | ----------- | ------------------------------- | ----------------------- |
|      `755` | `rwxr-xr-x` | Owner full, others read/execute | Scripts/directories     |
|      `644` | `rw-r--r--` | Owner read/write, others read   | Config/files            |
|      `700` | `rwx------` | Owner only                      | Private directories     |
|      `600` | `rw-------` | Owner only                      | Secrets/private keys    |
|      `777` | `rwxrwxrwx` | Everyone full access            | **Avoid in production** |

### Remember

```text
755 → rwx r-x r-x
644 → rw- r-- r--
700 → rwx ---
600 → rw- ---
777 → rwx rwx rwx
```

---

# 6. `chmod` — Change Permissions

### Numeric

```bash
chmod 755 script.sh
```

### Add Execute

```bash
chmod +x script.sh
```

### Remove Execute

```bash
chmod -x script.sh
```

### Owner Write

```bash
chmod u+w file
```

### Group Write

```bash
chmod g+w file
```

### Others Write

```bash
chmod o+w file
```

### Remove Others Write

```bash
chmod o-w file
```

### Permission Classes

```text
u → user/owner
g → group
o → others
a → all
```

---

# 7. `chown` — Change Ownership

```bash
chown user file
```

Change owner and group:

```bash
chown user:group file
```

Recursive:

```bash
chown -R user:group /directory
```

### Example

```bash
sudo chown -R appuser:appgroup /opt/myapp
```

---

# 8. `chgrp` — Change Group

```bash
chgrp group file
```

Example:

```bash
chgrp devops deploy.sh
```

---

# 9. Users and Groups

## Identify Current User

```bash
whoami
```

## Show User ID and Groups

```bash
id
```

## Specific User

```bash
id username
```

## Show Groups

```bash
groups username
```

---

# 10. User Management

### Create User

```bash
sudo useradd username
```

### Create User with Home Directory

```bash
sudo useradd -m username
```

### Set Password

```bash
sudo passwd username
```

### Delete User

```bash
sudo userdel username
```

### Add User to Group

```bash
sudo usermod -aG group user
```

### ⭐ Critical Interview Point

```bash
-aG
```

means **append the user to supplementary groups**.

Using:

```bash
-G
```

without `-a` replaces the existing supplementary groups.

### Interview Answer

> **"`-G` replaces supplementary groups, while `-aG` appends to the existing groups. I use `-aG` when adding a user to a group so existing memberships aren't removed."**

---

# 11. Group Management

### Create Group

```bash
sudo groupadd groupname
```

### Delete Group

```bash
sudo groupdel groupname
```

---

# 12. Important User/Permission Files

| File           | Purpose                     |
| -------------- | --------------------------- |
| `/etc/passwd`  | User account information    |
| `/etc/shadow`  | Password hashes; restricted |
| `/etc/group`   | Group information           |
| `/etc/sudoers` | sudo permissions            |

### Important

Edit sudo configuration using:

```bash
sudo visudo
```

rather than directly editing `/etc/sudoers`.

---

# 13. Symbolic Links

Create a symbolic link:

```bash
ln -s /var/log/app/app.log app.log
```

Result:

```text
app.log → /var/log/app/app.log
```

---

# 14. Symbolic Link vs Hard Link

| Feature           | Symbolic Link       | Hard Link             |
| ----------------- | ------------------- | --------------------- |
| Points to         | Path                | Same inode/data       |
| Target deleted    | Link becomes broken | Link still works      |
| Cross filesystem  | Yes                 | No                    |
| Directory linking | Possible            | Generally not allowed |

### Interview Answer

> **"A symbolic link points to a path, while a hard link points to the same inode. If the original file is deleted, a symbolic link becomes broken, while a hard link can still access the data."**

---

# 15. ACLs

ACLs allow more granular permissions than normal owner/group/others permissions.

### View ACL

```bash
getfacl file
```

### Grant Specific User Access

```bash
setfacl -m u:user:rwx file
```

### Remove ACL Entry

```bash
setfacl -x u:user file
```

### Use Case

Give one additional user access without changing the file's owner or primary group.

---

# 16. How Linux Decides File Access

Example:

```bash
-rw-r----- 1 appuser developers config.yml
```

Interpretation:

```text
Owner:  appuser
Group:  developers

appuser
  → rw-

developers member
  → r--

Everyone else
  → ---
```

### Critical Concept

Linux does **not** combine permissions from all three classes.

The applicable class determines the permission.

---

# 17. Permission Denied — Troubleshooting

Suppose an application returns:

```text
Permission denied
```

### ❌ Don't immediately do this

```bash
chmod 777 file
```

### Correct Troubleshooting Flow

```text
Permission denied
       ↓
Which user runs the application?
       ↓
Who owns the file?
       ↓
What group owns it?
       ↓
What permissions exist?
       ↓
Can the user traverse parent directories?
       ↓
Fix only the required permission/ownership
```

---

## Step 1 — Identify Application User

```bash
whoami
ps aux | grep app
```

---

## Step 2 — Check Ownership

```bash
ls -l file
```

Example:

```text
-rw-r----- appuser developers config.yml
```

---

## Step 3 — Check User's Groups

```bash
id
```

or:

```bash
id username
```

---

## Step 4 — Check Permissions

```bash
ls -l file
stat file
```

---

## Step 5 — Check Directory Traversal

```bash
ls -ld /path/to/dir
```

Remember:

> Directory `x` permission is required for traversal.

---

## Step 6 — Fix Only What Is Required

Example:

```bash
chown user:group file
chmod 640 file
```

Avoid blindly using:

```bash
chmod 777
```

---

# 18. Least Privilege

Give every user/process only the permissions it actually needs.

```text
Application user
      ↓
Required permissions only

Deployment group
      ↓
Required permissions only

Others
      ↓
No unnecessary access
```

### Production Principle

Instead of:

```bash
chmod 777
```

prefer something appropriate such as:

```bash
chmod 755
```

or:

```bash
chmod 640
```

with correct ownership.

---

# 19. High-Value Interview Questions

### Q1. What is `chmod 755`?

> **"`755` means the owner has read, write, and execute permissions, while group and others have read and execute permissions."**

```text
755
 ↓
7 → rwx → owner
5 → r-x → group
5 → r-x → others
```

---

### Q2. Difference between `755` and `777`?

> **"755 gives full access to the owner and read/execute access to group and others. 777 gives full access to everyone. I would avoid 777 in production because it violates least privilege."**

---

### Q3. Why can a user have read permission on a directory but still get Permission denied?

> **"Because directory execute permission is required to enter or traverse the directory."**

---

### Q4. Difference between `-G` and `-aG`?

> **"`-G` replaces supplementary group membership, while `-aG` appends to existing groups. I normally use `-aG` when adding a user to a group."**

---

### Q5. What is the first thing you do when you get Permission denied?

> **"I don't immediately use chmod 777. I first identify the process user, check ownership, group membership, permissions, and directory traversal permissions, then apply the minimum required fix."**

---

### Q6. How do you change owner and group?

```bash
chown user:group file
```

---

### Q7. How do you recursively change ownership?

```bash
chown -R user:group /directory
```

---

### Q8. What is an ACL?

> **"ACL provides fine-grained access control, allowing permissions to be granted to specific users or groups beyond the normal owner/group/others model."**

---

# 20. Commands to Memorize

```bash
ls -l
chmod
chown
chgrp
stat
find
whoami
id
groups
useradd
usermod
passwd
groupadd
groupdel
sudo
ln -s
getfacl
setfacl
```

---

# ⭐ Interview Revision Sheet

```text
r = 4
w = 2
x = 1

755 = rwxr-xr-x
644 = rw-r--r--
700 = rwx------
600 = rw-------

u = owner
g = group
o = others

chmod → permissions
chown → owner + group
chgrp → group
usermod -aG → append group
ls -l → permissions + ownership
id → UID/GID/groups
getfacl → view ACL
setfacl → modify ACL

Directory:
r → list
w → create/delete/rename
x → enter/traverse

Golden rule:
DON'T use chmod 777 blindly.
Find user → group → owner → permissions → traversal → fix minimally.
```

> **If you can explain `ls -l`, calculate `755/644/600`, troubleshoot `Permission denied`, explain `chmod/chown/usermod -aG`, and distinguish symbolic vs hard links, Section 2 is interview-ready.**
