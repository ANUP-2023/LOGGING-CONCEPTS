# FastAPI Deployment with OS-level Log Rotation (logrotate)
A from-scratch, production-style guide to running FastAPI with **system-maintained logs** using logrotate (not Python rotation). This keeps your logs tidy, compressed, and handled by the OS—ideal for robust, long-term services.

---

## 1. Install Basics

```bash
sudo apt update
sudo apt install python3-pip python3-venv -y
```

---

## 2. Create Project Structure

```bash
sudo mkdir -p /opt/app
cd /opt/app

python3 -m venv venv
source venv/bin/activate

pip install fastapi uvicorn
```

---

## 3. FastAPI App with Plain File Logging

Create `main.py` with the following (no rotation handler—rotation is handled by logrotate):

```python
import os
import logging
from fastapi import FastAPI

LOG_DIR = "/var/log/app"
LOG_FILE = os.path.join(LOG_DIR, "app.log")

os.makedirs(LOG_DIR, exist_ok=True)

logger = logging.getLogger("fastapi_app")
logger.setLevel(logging.INFO)

handler = logging.FileHandler(LOG_FILE)
formatter = logging.Formatter("%(asctime)s | %(levelname)s | %(message)s")
handler.setFormatter(formatter)
logger.addHandler(handler)

app = FastAPI()

@app.get("/")
async def home():
    logger.info("Home endpoint hit")
    return {"message": "Hello World"}
```

---

## 4. Create Log Directory & User

```bash
sudo mkdir -p /var/log/app
sudo useradd --system --no-create-home --shell /usr/sbin/nologin appuser
sudo chown -R appuser:appuser /opt/app
sudo chown -R appuser:appuser /var/log/app
```

---

## 5. Test Locally

```bash
source /opt/app/venv/bin/activate
uvicorn main:app --host 0.0.0.0 --port 8000
tail -f /var/log/app/app.log
```

---

## 6. Create a systemd Service

```bash
sudo nano /etc/systemd/system/app.service
```

Paste:

```ini
[Unit]
Description=FastAPI App
After=network.target

[Service]
User=appuser
Group=appuser
WorkingDirectory=/opt/app
ExecStart=/opt/app/venv/bin/uvicorn main:app --host 0.0.0.0 --port 8000

Restart=always

[Install]
WantedBy=multi-user.target
```

---

## 7. Setup logrotate for FastAPI

Create config file `/etc/logrotate.d/app`:

```bash
sudo nano /etc/logrotate.d/app
```

Paste:

```conf
/var/log/app/app.log {
    daily
    missingok
    rotate 7
    compress
    delaycompress
    notifempty
    create 0640 appuser appuser
    copytruncate
}
```
- `copytruncate` allows FastAPI to keep writing, avoiding a restart.
- `compress`/`delaycompress` will gzip older logs one rotation after they're created.
- `rotate 7` keeps one week of logs.

---

## 8. Test Log Rotation

Force a rotation and check the logs:

```bash
sudo logrotate -f /etc/logrotate.d/app
ls -l /var/log/app
```

You should see:
```
app.log
app.log.1
app.log.2.gz
```

---

## 9. (Optional) Fine-tuned Scheduling with Cron

To run logrotate at 2:00 AM daily:

```bash
sudo crontab -e
```

Add:

```
0 2 * * * /usr/sbin/logrotate /etc/logrotate.conf
```

---

## 10. Troubleshooting and Tips

- If both cron and systemd timer are active, you may get duplicate rotations. Disable one:
    ```bash
    sudo systemctl disable logrotate.timer
    ```
- To debug:
    ```bash
    grep logrotate /var/log/syslog
    ```
- `copytruncate` can rarely cause minor log loss under extreme loads, but is safe for most FastAPI usage.

---

## **Summary**
- **FastAPI** writes to `/var/log/app/app.log` with a plain file handler.
- **logrotate** daily rotates and compresses logs.
- **cron** or **systemd timer** triggers rotation.

This setup leverages the OS for robust, reliable log management—no Python rotation logic needed!
