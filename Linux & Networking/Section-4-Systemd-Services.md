# Linux Section 4: Systemd & Services

> **Interview Goal:** Understand `systemd`, service lifecycle, `systemctl`, service files, `journalctl`, targets, systemd timers, and how to troubleshoot failed services.

---

# 1. What is systemd?

`systemd` is the modern Linux **system and service manager**.

It is typically:

* PID 1
* Responsible for system initialization
* Responsible for managing services/daemons
* Responsible for service lifecycle
* Integrated with `journald` for logging

### Interview Answer

> **"systemd is the system and service manager on modern Linux. It runs as PID 1 and manages system initialization, services, their lifecycle, and related logging."**

---

# 2. `systemctl`

`systemctl` is the main command used to interact with systemd.

---

# 3. Start / Stop / Restart / Status

### Start a service

```bash
sudo systemctl start nginx
```

### Stop a service

```bash
sudo systemctl stop nginx
```

### Restart a service

```bash
sudo systemctl restart nginx
```

### Check status

```bash
sudo systemctl status nginx
```

### Reload configuration

```bash
sudo systemctl reload nginx
```

---

# 4. `restart` vs `reload`

This is a **very common interview question**.

| Command   | Behavior                                       | Use                   |
| --------- | ---------------------------------------------- | --------------------- |
| `restart` | Stops and starts process                       | Full process restart  |
| `reload`  | Reloads configuration without stopping process | Configuration changes |

### Example

You modify nginx configuration:

```bash
sudo systemctl reload nginx
```

This attempts to apply the configuration without fully stopping the service.

If you need a full restart:

```bash
sudo systemctl restart nginx
```

### Interview Answer

> **"`reload` applies configuration changes without stopping the service, while `restart` completely stops and starts the service. I prefer reload when the service supports it and only configuration needs to change."**

---

# 5. Enable vs Disable

These control whether a service starts automatically during boot.

### Enable

```bash
sudo systemctl enable nginx
```

Means:

```text
Start nginx automatically at boot
```

### Disable

```bash
sudo systemctl disable nginx
```

Means:

```text
Don't start nginx automatically at boot
```

Check:

```bash
systemctl is-enabled nginx
```

---

# 6. `disable` vs `mask`

Another common interview question.

| Command   | Meaning                                        |
| --------- | ---------------------------------------------- |
| `disable` | Prevents automatic startup at boot             |
| `mask`    | Completely prevents service from being started |
| `unmask`  | Removes the mask                               |

### Disable

```bash
sudo systemctl disable nginx
```

You can still manually start it:

```bash
sudo systemctl start nginx
```

### Mask

```bash
sudo systemctl mask nginx
```

Now starting it manually is blocked.

### Unmask

```bash
sudo systemctl unmask nginx
```

### Interview Answer

> **"`disable` prevents automatic startup at boot but still allows manual startup. `mask` completely prevents the service from being started, even manually."**

---

# 7. Check Service State

### Is service active?

```bash
systemctl is-active nginx
```

Possible result:

```text
active
```

or:

```text
inactive
```

### Is service enabled?

```bash
systemctl is-enabled nginx
```

### Is service failed?

```bash
systemctl is-failed nginx
```

---

# 8. Service Files

Systemd services are defined using **unit files**.

Common locations:

```text
/usr/lib/systemd/system/
```

Default/system-provided service definitions.

```text
/etc/systemd/system/
```

Custom services and overrides.

### Important

Prefer creating custom/override configuration under:

```text
/etc/systemd/system/
```

rather than modifying vendor-provided files directly.

---

# 9. View a Service File

```bash
systemctl cat nginx
```

Or:

```bash
cat /usr/lib/systemd/system/nginx.service
```

---

# 10. Override a Service

Use:

```bash
sudo systemctl edit nginx
```

This creates an override configuration under:

```text
/etc/systemd/system/
```

### Why Overrides Matter

You can customize things such as:

* Environment variables
* Restart behavior
* Resource limits
* Commands
* Dependencies

without modifying the original vendor service file.

---

# 11. Systemd Unit File Structure

Example:

```ini
[Unit]
Description=My application
After=network.target

[Service]
Type=simple
User=appuser
Group=appgroup
WorkingDirectory=/opt/app
ExecStart=/usr/bin/python3 /opt/app/main.py
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

---

# 12. `[Unit]`

Contains metadata and dependency/order information.

### Description

```ini
Description=My application
```

Human-readable description.

### After

```ini
After=network.target
```

Controls startup ordering.

It means the service should be started after the specified unit.

> **Important:** `After=` controls ordering; it does not by itself create a dependency.

---

# 13. `[Service]`

Defines how the application runs.

### Type

```ini
Type=simple
```

### User

```ini
User=appuser
```

Runs the service as that user.

### Group

```ini
Group=appgroup
```

### WorkingDirectory

```ini
WorkingDirectory=/opt/app
```

### ExecStart

```ini
ExecStart=/usr/bin/python3 /opt/app/main.py
```

Defines the command used to start the service.

### Restart

```ini
Restart=always
```

Controls restart behavior.

### RestartSec

```ini
RestartSec=10
```

Wait 10 seconds before restarting.

---

# 14. `[Install]`

Defines how the service is enabled.

Example:

```ini
[Install]
WantedBy=multi-user.target
```

This means the service is associated with the `multi-user.target` when enabled.

---

# 15. Important Service Types

| Type      | When systemd considers service started   | Typical Use                 |
| --------- | ---------------------------------------- | --------------------------- |
| `simple`  | When `ExecStart` begins                  | Most services               |
| `forking` | Parent process exits after forking       | Traditional daemons         |
| `oneshot` | After command finishes                   | One-time tasks              |
| `notify`  | Application sends readiness notification | Apps supporting `sd_notify` |

### Most Important

```ini
Type=simple
```

is common for normal long-running applications.

---

# 16. `journalctl` — Systemd Logs

`journalctl` is used to inspect systemd/journald logs.

### All logs for service

```bash
journalctl -u nginx
```

### Follow logs live

```bash
journalctl -u nginx -f
```

Similar idea to:

```bash
tail -f
```

### Last 50 lines

```bash
journalctl -u nginx -n 50
```

### Today's logs

```bash
journalctl -u nginx --since today
```

### Last hour

```bash
journalctl -u nginx --since "1 hour ago"
```

### Current boot

```bash
journalctl -b
```

### Previous boot

```bash
journalctl -b -1
```

---

# 17. `systemctl status` + `journalctl`

This is an extremely useful troubleshooting combination.

```bash
sudo systemctl status myapp
```

Then:

```bash
sudo journalctl -u myapp -n 50
```

For live logs:

```bash
sudo journalctl -u myapp -f
```

### Interview Answer

> **"I first check `systemctl status` to understand the service state and recent failure information, then use `journalctl -u` to inspect the detailed service logs."**

---

# 18. Systemd Targets

Targets are groups of units representing system states.

| Target              | Traditional Runlevel | Purpose                           |
| ------------------- | -------------------: | --------------------------------- |
| `poweroff.target`   |                    0 | Shutdown                          |
| `rescue.target`     |                    1 | Single-user/rescue mode           |
| `multi-user.target` |                    3 | Multi-user, generally without GUI |
| `graphical.target`  |                    5 | Graphical environment             |
| `reboot.target`     |                    6 | Reboot                            |

### Check Default Target

```bash
systemctl get-default
```

### Change Default Target

```bash
sudo systemctl set-default multi-user.target
```

---

# 19. Systemd Timers

Systemd timers can be used for scheduled tasks.

They provide functionality similar to traditional cron jobs.

A timer commonly works together with a service.

```text
backup.timer
      ↓
backup.service
      ↓
backup.sh
```

---

# 20. Example Timer

File:

```text
/etc/systemd/system/backup.timer
```

```ini
[Unit]
Description=Daily backup timer

[Timer]
OnCalendar=daily
Persistent=true

[Install]
WantedBy=timers.target
```

---

# 21. Timer Service

File:

```text
/etc/systemd/system/backup.service
```

```ini
[Unit]
Description=Backup service

[Service]
Type=oneshot
ExecStart=/usr/local/bin/backup.sh
```

The timer triggers the service.

---

# 22. Enable and Start Timer

```bash
sudo systemctl enable backup.timer
sudo systemctl start backup.timer
```

List timers:

```bash
systemctl list-timers
```

---

# 23. Useful Systemd Commands

### List services

```bash
systemctl list-units --type=service
```

### Show failed services

```bash
systemctl --failed
```

### Show dependencies

```bash
systemctl list-dependencies nginx
```

### Show service file

```bash
systemctl cat nginx
```

---

# 24. Troubleshooting: Service Isn't Running

Suppose:

```text
myapp.service
```

is not running.

### Step 1 — Check status

```bash
sudo systemctl status myapp
```

### Step 2 — Check logs

```bash
sudo journalctl -u myapp -n 50
```

### Step 3 — Try starting it

```bash
sudo systemctl start myapp
```

### Step 4 — Check again

```bash
sudo systemctl status myapp
```

### Troubleshooting Flow

```text
Service not running
       ↓
systemctl status
       ↓
What error?
       ↓
journalctl -u
       ↓
Identify root cause
       ↓
Fix configuration/application/dependency
       ↓
systemctl restart/start
       ↓
Verify status
```

---

# 25. Troubleshooting: Service Keeps Crashing

### Step 1

```bash
sudo systemctl status myapp
```

### Step 2

```bash
sudo journalctl -u myapp -f
```

### Step 3

Check restart configuration:

```bash
sudo systemctl cat myapp | grep Restart
```

### Step 4

If necessary:

```bash
sudo systemctl stop myapp
```

Then investigate the actual failure.

### Don't blindly keep restarting

A service repeatedly crashing usually indicates an underlying problem such as:

* Bad configuration
* Missing dependency
* Permission issue
* Application crash
* Invalid environment variable
* Port conflict

---

# 26. Troubleshooting: Service Not Starting at Boot

Check:

```bash
sudo systemctl is-enabled myapp
```

If disabled:

```bash
sudo systemctl enable myapp
```

Then verify:

```bash
systemctl is-enabled myapp
```

---

# 27. Creating a Production Service — Basic Pattern

Example:

```ini
[Unit]
Description=My Application
After=network.target

[Service]
Type=simple
User=appuser
Group=appgroup
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/python3 /opt/myapp/main.py
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Then:

```bash
sudo systemctl daemon-reload
sudo systemctl enable myapp
sudo systemctl start myapp
sudo systemctl status myapp
```

### Important

After changing unit files:

```bash
sudo systemctl daemon-reload
```

is required so systemd reloads the unit configuration.

---

# 28. High-Value Interview Questions

### Q1. What is systemd?

> **"systemd is the system and service manager on modern Linux. It runs as PID 1 and manages system initialization, services, their lifecycle, and logging through journald."**

---

### Q2. How do you check whether a service is running?

```bash
sudo systemctl status nginx
```

or:

```bash
systemctl is-active nginx
```

---

### Q3. `restart` vs `reload`?

> **"`restart` completely stops and starts the service. `reload` applies configuration changes without stopping the service, when supported."**

---

### Q4. `enable` vs `start`?

> **"`start` starts the service now. `enable` configures the service to start automatically at boot."**

This distinction is **very important**.

```bash
systemctl start nginx
```

→ Start now.

```bash
systemctl enable nginx
```

→ Start automatically at boot.

You can also do:

```bash
sudo systemctl enable --now nginx
```

→ Enable for boot **and start immediately**.

---

### Q5. `disable` vs `mask`?

> **"`disable` prevents automatic startup at boot but allows manual startup. `mask` completely prevents the service from being started."**

---

### Q6. How do you check real-time service logs?

```bash
journalctl -u myapp -f
```

---

### Q7. How do you find failed services?

```bash
systemctl --failed
```

---

### Q8. Where are systemd service files?

> **"System-provided service files commonly exist under `/usr/lib/systemd/system/`, while custom services and overrides are commonly placed under `/etc/systemd/system/`."**

---

### Q9. Why use `systemctl edit`?

> **"It creates an override configuration so I can customize a service without directly modifying the original vendor-provided unit file."**

---

### Q10. What does `ExecStart` do?

> **"`ExecStart` defines the command systemd executes to start the service."**

---

### Q11. Why use `Restart=on-failure`?

> **"It tells systemd to automatically restart the service when it exits due to a failure."**

---

### Q12. What is `journalctl`?

> **"`journalctl` is the command-line tool used to query logs collected by systemd's journal."**

---

# ⭐ Interview Revision Sheet

```text
systemd
  ↓
PID 1
  ↓
System + service manager

systemctl start
  → Start NOW

systemctl stop
  → Stop NOW

systemctl restart
  → Stop + start

systemctl reload
  → Reload configuration

systemctl enable
  → Start automatically at boot

systemctl disable
  → Don't start automatically at boot

systemctl mask
  → Completely prevent starting

systemctl status
  → Service status

systemctl is-active
  → Is it running?

systemctl is-enabled
  → Starts at boot?

systemctl --failed
  → Failed services

journalctl -u SERVICE
  → Service logs

journalctl -u SERVICE -f
  → Live service logs

systemctl cat SERVICE
  → View service definition

systemctl edit SERVICE
  → Create override

daemon-reload
  → Reload changed unit files
```

---

# 🔥 Service Troubleshooting Cheat Sheet

```bash
# 1. Check status
sudo systemctl status myapp

# 2. Check logs
sudo journalctl -u myapp -n 50

# 3. Follow logs
sudo journalctl -u myapp -f

# 4. Check whether enabled
systemctl is-enabled myapp

# 5. Check whether active
systemctl is-active myapp

# 6. Check failed services
systemctl --failed

# 7. View configuration
systemctl cat myapp

# 8. Reload unit configuration after editing
sudo systemctl daemon-reload

# 9. Restart
sudo systemctl restart myapp

# 10. Enable at boot
sudo systemctl enable myapp
```

---

# ⭐ Must-Know Concepts

```text
systemd       → Service/system manager
PID 1         → systemd on typical modern Linux
systemctl     → Manage services
journalctl    → Read systemd logs

start         → Start now
enable        → Start at boot
restart       → Full restart
reload        → Reload config
disable       → Disable boot startup
mask          → Block service completely

[Unit]        → Metadata/dependencies/order
[Service]     → How application runs
[Install]     → Enablement/target

ExecStart     → Start command
User          → Service user
Group         → Service group
WorkingDirectory
              → Working directory
Restart       → Restart policy
RestartSec    → Delay before restart
WantedBy      → Target association

/usr/lib/systemd/system
              → Vendor/system units

/etc/systemd/system
              → Custom/override units
```

> **Section 4 is interview-ready when you can troubleshoot a failed service from `systemctl status` → `journalctl` → service configuration → dependency/configuration fix, and clearly explain `start`, `enable`, `restart`, `reload`, `disable`, and `mask`.**
