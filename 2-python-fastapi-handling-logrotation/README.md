# FastAPI Daily Log-Rotating Deployment Guide

A practical, production-ready from-scratch setup to run a FastAPI app on Ubuntu with **daily, auto-rotated logs**. This guide uses Python's `TimedRotatingFileHandler` for simple Python-driven log management.

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

## 3. Create FastAPI App with Daily Logs

Create `main.py` with the following code:

```python
import os
import logging
from logging.handlers import TimedRotatingFileHandler
from fastapi import FastAPI

LOG_DIR = "/var/log/app"
LOG_FILE = os.path.join(LOG_DIR, "app.log")

# Ensure log directory exists
os.makedirs(LOG_DIR, exist_ok=True)

# Logger setup
logger = logging.getLogger("fastapi_app")
logger.setLevel(logging.INFO)

handler = TimedRotatingFileHandler(
    LOG_FILE,
    when="midnight",   # Rotate daily
    interval=1,
    backupCount=7      # Keep 7 days of logs
)

formatter = logging.Formatter(
    "%(asctime)s | %(levelname)s | %(message)s"
)
handler.setFormatter(formatter)

logger.addHandler(handler)

app = FastAPI()

@app.on_event("startup")
async def startup():
    logger.info("Application started")

@app.get("/")
async def home():
    logger.info("Home endpoint hit")
    return {"message": "Hello World"}

@app.get("/error")
async def error():
    try:
        1 / 0
    except Exception:
        logger.exception("Error occurred")
        return {"error": "Something went wrong"}

@app.get("/health")
async def health():
    return {"status": "ok"}
```

---

## 4. Create Log Directory (IMPORTANT)

```bash
sudo mkdir -p /var/log/app
```

---

## 5. Create a Dedicated User (Recommended)

```bash
sudo useradd --system --no-create-home --shell /usr/sbin/nologin appuser
```

Give permissions:

```bash
sudo chown -R appuser:appuser /opt/app
sudo chown -R appuser:appuser /var/log/app
```

---

## 6. Test Manually

```bash
source /opt/app/venv/bin/activate
uvicorn main:app --host 0.0.0.0 --port 8000
```

Check logs:

```bash
tail -f /var/log/app/app.log
```

---

## 7. Create systemd Service

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

## 8. Start Service

```bash
sudo systemctl daemon-reload
sudo systemctl enable app
sudo systemctl start app
```

Check status:

```bash
systemctl status app
```

---

## 9. Verify Daily Log Rotation

Your logs should look like:

```
/var/log/app/app.log
/var/log/app/app.log.2026-04-22
/var/log/app/app.log.2026-04-21
```

- Rotates at **midnight**
- Keeps **7 days**
- Old logs automatically deleted

---

## 10. (Optional) Add Compression

Enhance logging by compressing old logs:

Add this below handler in your code:

```python
handler.namer = lambda name: name + ".gz"

def rotator(source, dest):
    import gzip, shutil
    with open(source, "rb") as sf, gzip.open(dest, "wb") as df:
        shutil.copyfileobj(sf, df)
    os.remove(source)

handler.rotator = rotator
```

---

## Key Decisions Explained

- Using Python logging instead of logrotate → simpler, self-contained
- Using `systemd` → ensures auto-restart & boot startup
- Dedicated user → avoids permission headaches and improves security

---

## Common Pitfalls (Worth Avoiding)

- ❌ Not giving `/var/log/app` correct permissions → logs silently fail
- ❌ Running service as root → unnecessary risk
- ❌ Using both logrotate + Python rotation → causes conflicts

---

## Next Steps (Optional Improvements)

- **Gunicorn + multiple workers (real production setup)**
- **JSON structured logging for ELK / Loki**
- **Centralized logging (Grafana stack)**

---

**Note:** Python already rotates logs using the built-in `TimedRotatingFileHandler`. If you want to use OS-level rotation with logrotate, make sure to disable Python rotation to avoid conflicts.

