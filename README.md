# Logrotate: Automated Log File Management

Efficient log management is critical for maintaining the stability, security, and integrity of Linux systems. Without proper rotation, log files can grow uncontrollably, consuming valuable disk space and potentially degrading system performance. `logrotate` is a robust tool designed to automate log rotation, compression, and removal, keeping your systems clean and predictable.

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
  Directory for per-service log rotation policies. Add your own configs here for apps not covered by default.

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

