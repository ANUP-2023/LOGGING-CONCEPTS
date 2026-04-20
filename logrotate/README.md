# Why log rotation? 📈

Log rotation is essential to prevent log files from growing uncontrollably—it's one of the most basic but critical parts of running any production service.

🧠 **Core idea:**  
Logs grow forever → rotation is how you keep them under control.

---

### 🎯 Main reasons for log rotation

**💾 1. Prevent disk from filling up**

- If you don’t rotate:
  - `app.log` just keeps growing → can fill disk ❌
  - When disk is full:
    - Applications might crash or behave unpredictably
    - OS and services may stop writing logs (which is even worse)

**⚡ 2. Improve performance**

- Giant log files (tens of GBs):
  - Are slow to view/open (e.g. with `less`, `grep`)
  - Cause log tools to slow down
- Rotation keeps log files small and searchable

**🔍 3. Easier debugging**

- Instead of one huge log file (`app.log` = 10GB 😬), you get:
  - `app.log-20240420`
  - `app.log-20240419`
  - `app.log-20240418`
- Makes it faster to check logs for a specific date or incident

**🗜️ 4. Save storage via compression**

- Old logs are usually compressed (e.g. `.gz`, `.zst`)
- Cuts storage usage (especially useful on EFS/cloud storage)

**🧾 5. Retention & compliance**

- You can keep only the last X days/months/years of logs (as needed for audits, compliance, or investigations)
  - e.g. 7 days, 30 days, 1 year

**🔄 6. Works better with log aggregation tools**

- Logging systems (e.g. Grafana Loki) prefer:
  - Lots of small, rotated files over giant monolithic logs
  - Predictable naming/structure

**🚀 7. Keeps your system healthy**

- Without rotation:
  - `/logs` fills up → disk full → system crash
- With rotation:
  - Logs are controlled → system stays stable ✅

---

🧠 **Real-world example:**  
With Docker + EFS, if you don’t rotate, logs grow endlessly on your (potentially expensive) network storage and can degrade performance.  
With rotation: logs are split daily, compressed, and cleaned up after your set retention period.

---

## 🚀 Overview

`logrotate` is a utility that simplifies the management of log files—automatically rotating, compressing, and deleting logs based on customizable policies. It is widely used across Linux distributions and is typically run daily via `cron` or systemd timers.

---

## 📑 Table of Contents

- [Features](#features)
- [How Logrotate Works](#how-logrotate-works)
- [Key Configuration Files](#key-configuration-files)
- [Sample Logrotate Configuration](#sample-logrotate-configuration)
- [Rotation Triggers & Options](#rotation-triggers--options)
- [Usage](#usage)
- [References](#references)

---

## ✨ Features

- **Automated Rotation**: Archive old logs and create fresh, empty log files.
- **Compression**: Reduce disk usage by compressing rotated logs (`gzip`, `zstd`, etc.).
- **Retention Control**: Keep logs for a defined period or count before automatic deletion.
- **Flexible Scheduling**: Rotate logs based on time, size, or event-based triggers.
- **Custom Actions**: Execute scripts before or after rotation (e.g., reload daemons).
- **Fine-Grained Control**: Per-service/application policies via drop-in configs.

---

## ⚙️ How Logrotate Works

The typical log rotation cycle involves:

1. **Archive**: Rename the current log file (e.g., `app.log` → `app.log.1`).
2. **Create**: Generate a new log file with the original name so logging continues (`app.log`).
3. **Compress**: Optionally compress older logs (e.g., `app.log.2.gz`).
4. **Delete**: Remove the oldest logs beyond your retention policy (e.g., keep only last 7).
5. **Post-Processing**: Run custom commands if required.

---

## 🗂️ Key Configuration Files

- **`/etc/logrotate.conf`**  
  The main configuration file, often includes other configs.

- **`/etc/logrotate.d/`**  
  Directory for per-service log rotation policies. For example, you might add files like `app1-logs`, `app2-logs`, or `app3-logs` here—each defining rotation rules for its respective application's logs. Add your own configs in this directory for apps not covered by default.

---

## 📝 Sample Logrotate Configuration

Below is an example `logrotate` policy for `/var/log/app/app.log`:

```conf
/var/log/app/app.log {
    # Rotate daily
    daily

    # Do not throw error if file is missing
    missingok

    # Keep 365 rotations (1 year)
    rotate 365

    # Compress old logs with zstd
    compress
    compresscmd /usr/bin/zstd
    uncompresscmd /usr/bin/unzstd

    # Don't rotate if log is empty
    notifempty

    # Create a new file each rotation with permissions 0644, owner root
    create 0644 root root

    # Add date to rotated logs (e.g., app.log-20260420)
    dateext

    # Run postrotate script (add actions as needed)
    sharedscripts
    postrotate
        # Example: systemctl reload your-app
        # kill -HUP <pid>
    endscript
}
```

---

## ⏳ Rotation Triggers & Options

- **Time-based**: `daily`, `weekly`, `monthly`
- **Size-based**: Rotate when log exceeds a size (e.g., `maxsize 100M`)
- **Count-based**: Retain last _N_ archives (`rotate 7`)
- **Compression**: `compress` (default gzip), `compresscmd` for custom compressor
- **Empty-File Handling**: `notifempty` (skip empty logs)
- **Delayed Compression**: `delaycompress` (compress one cycle later)
- **Copy & Truncate**: `copytruncate` (if app can't close handles)

View a full list of directives in the [`logrotate` man page](https://man7.org/linux/man-pages/man8/logrotate.8.html).

---

## 🛠️ Usage

- **Find Logrotate Binary:**
  ```sh
  which logrotate
  # /usr/sbin/logrotate
  ```

There are two ways to run logrotate:

1. **Manual Execution:**  
   You can manually trigger logrotate with various options:
   - To preview what would be done (dry run):
     ```sh
     logrotate -d /etc/logrotate.conf
     ```
   - To force log rotation immediately:
     ```sh
     logrotate -f /etc/logrotate.conf
     ```
   - For detailed output:
     ```sh
     logrotate -v /etc/logrotate.conf
     ```
   - To specify a custom state file:
     ```sh
     logrotate -s /var/lib/logrotate.status
     ```

2. **Automatic Execution:**  
   Logrotate can be automatically scheduled using either cron jobs or systemd timers, for example:
   ```sh
   /usr/sbin/logrotate /etc/logrotate.conf
   ```

---

## 📝 References

- [logrotate man page](https://man7.org/linux/man-pages/man8/logrotate.8.html)
- [Linux Die.net: logrotate](https://linux.die.net/man/8/logrotate)
- [Sample logrotate configs](https://github.com/search?q=logrotate.conf)

---

With an effective `logrotate` setup, you'll ensure logs never grow out of control, maintain system health, and comply with security or retention policies. Customize your configuration in `/etc/logrotate.d/` for best results!

---

