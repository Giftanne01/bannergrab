# ⚡ BannerGrab
**[▶ Live Demo](https://YOURUSERNAME.github.io/bannergrab)**
```
╔══════════════════════════════════════╗
║  BANNERGRAB v1.0 — Service Recon    ║
║  Python 3 · Zero Dependencies       ║
╚══════════════════════════════════════╝
```

> A lightweight network service fingerprinting tool written in Python.  
> Connects to a target host and retrieves the **service banner** — the raw response most daemons send on connection — to identify software, versions, and configurations.

![Python](https://img.shields.io/badge/Python-3.8+-00cc33?style=flat-square&logo=python&logoColor=white&labelColor=0a0c0f)
![License](https://img.shields.io/badge/License-MIT-00ff41?style=flat-square&labelColor=0a0c0f)
![Dependencies](https://img.shields.io/badge/Dependencies-None-00ff41?style=flat-square&labelColor=0a0c0f)
![Status](https://img.shields.io/badge/Status-Active-00cc33?style=flat-square&labelColor=0a0c0f)

---

## What It Does

Banner grabbing is a core recon technique in network security. When a service starts a TCP session, it often immediately broadcasts its identity — software name, version, OS details. BannerGrab automates this process cleanly and efficiently.

**Example output:**

```
[*] Connecting to 192.168.1.10:22...
[+] Banner received:
    SSH-2.0-OpenSSH_8.9p1 Ubuntu-3ubuntu0.6
```

From that single line you can infer: OpenSSH version, the OS distribution, and a patch level — all without authentication.

---

## Features

| Feature | Detail |
|---|---|
| **Passive mode** | Listens on connect — works for SSH, FTP, SMTP, Redis |
| **Active probe support** | Sends HTTP HEAD, SMTP EHLO when services don't speak first |
| **Configurable timeout** | Per-scan socket timeout via argument |
| **Clean CLI** | Minimal, readable output optimised for scripting |
| **Zero dependencies** | Pure Python standard library only |
| **Extendable** | Easy to add new probe types or batch scanning |

---

## Quick Start

**Requirements:** Python 3.8+, no pip installs needed.

```bash
git clone https://github.com/yourname/bannergrab.git
cd bannergrab
```

**Run interactively:**
```bash
python3 banner_grabber.py
```

**Run with arguments:**
```bash
python3 banner_grabber.py <ip> <port>
```

**Examples:**
```bash
# SSH fingerprinting
python3 banner_grabber.py 192.168.1.1 22
# → SSH-2.0-OpenSSH_8.9p1 Ubuntu-3ubuntu0.6

# FTP server identification
python3 banner_grabber.py 10.0.0.5 21
# → 220 ProFTPD 1.3.5 Server (Debian)

# Database version check
python3 banner_grabber.py 10.0.0.5 3306
# → 5.7.42-log MySQL Community Server

# Interactive mode
python3 banner_grabber.py
Enter target IP: 192.168.1.1
Enter target port: 80
```

---

## The Code

```python
import socket

def grab_banner(ip, port, timeout=2):
    try:
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.settimeout(timeout)
        s.connect((ip, port))
        banner = s.recv(1024).decode('utf-8', errors='replace').strip()
        s.close()
        return banner
    except socket.timeout:
        return None
    except socket.error:
        return None
```

Simple by design. The core logic is ~10 lines. The interesting work is in understanding *what to do with the output*.

---

## Supported Services (Demo Mode)

| Port | Service | Banner Pattern |
|------|---------|---------------|
| 21 | FTP | `220 ProFTPD ...` |
| 22 | SSH | `SSH-2.0-OpenSSH_x.x ...` |
| 25 | SMTP | `220 ... ESMTP Postfix` |
| 80/443 | HTTP/S | `HTTP/1.1 200 OK\nServer: ...` |
| 110 | POP3 | `+OK Dovecot ready` |
| 3306 | MySQL | Version string |
| 5432 | PostgreSQL | Error includes version info |
| 6379 | Redis | `+PONG` |

---

## Extending It

**Scan multiple ports:**
```python
PORTS = [21, 22, 25, 80, 443, 3306, 5432, 6379]

for port in PORTS:
    banner = grab_banner(target_ip, port)
    tag = f"[+] {port:5d}" if banner else f"[-] {port:5d}"
    print(f"{tag} : {banner or 'no response'}")
```

**Add an HTTP probe** (services that don't speak first):
```python
try:
    banner = s.recv(1024)
except socket.timeout:
    s.send(b"HEAD / HTTP/1.0\r\n\r\n")
    banner = s.recv(1024)
```

**Write results to file:**
```python
with open("scan_results.txt", "a") as f:
    f.write(f"{ip}:{port} -> {banner}\n")
```

---

## Project Structure

```
bannergrab/
├── banner_grabber.py    # Core tool
├── README.md            # Documentation
└── LICENSE              # MIT
```

---

## Security & Ethics

This tool is for **authorized use only**.

- Your own servers and home lab
- CTF competitions and sandboxed environments
- Authorized penetration tests (written permission)

> Unauthorized scanning may violate the Computer Fraud and Abuse Act (CFAA), the Computer Misuse Act (CMA), or equivalent laws in your jurisdiction.

---

## What I Learned Building This

- How TCP socket lifecycles work at the stdlib level in Python
- The difference between services that broadcast vs. services that wait for a probe
- Why banner grabbing is step one of any real service enumeration workflow
- How version strings leak more information than most sysadmins realize

---

## License

MIT — free to use, modify, and distribute with attribution.

---

*Built from scratch · Pure Python stdlib · No pip required*
