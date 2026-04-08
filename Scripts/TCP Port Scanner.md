# 🔍 TCP Port Scanner

> Python-based TCP port scanner for network reconnaissance. Supports flexible port input (single, range, mixed), multithreaded scanning, service identification, and banner grabbing. Built as part of hands-on offensive security tooling practice.

![Python](https://img.shields.io/badge/Python-3.10%2B-blue?style=flat-square&logo=python)
![Category](https://img.shields.io/badge/Category-Network%20Recon-red?style=flat-square)
![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)
![Status](https://img.shields.io/badge/Status-Active-brightgreen?style=flat-square)

---

## 📌 Overview

This tool performs fast TCP connect scans against a target host across a user-defined set of ports. It is designed for use during the **reconnaissance phase** of a penetration test or network audit, helping identify open services, exposed ports, and potential attack surface.

Built with Python's standard library — no pip installs required.

---

## ✨ Features

| Feature | Details |
|---|---|
| 🔌 TCP Connect Scan | Full TCP handshake to confirm port state |
| ⚡ Multithreaded | Concurrent scanning via `ThreadPoolExecutor` |
| 🎯 Flexible Port Input | Single, comma-separated, ranges, or mixed |
| 🏷️ Service Detection | IANA-based service name resolution |
| 📢 Banner Grabbing | Captures raw service banners on connect |
| 🎨 Coloured Output | OPEN/closed/error states in colour |
| 📋 Summary Report | Clean open-port summary with elapsed time |

---

## 🛠️ Requirements

- Python 3.10+
- No external dependencies
- `colorama` *(optional — for colour support on Windows)*

```bash
# Optional colour support on Windows
pip install colorama
```

---

## 🚀 Usage

### Interactive Mode
```bash
python3 port_scanner.py
```
You will be prompted for a target and port list.

### CLI Mode

```bash
# Scan specific ports
python3 port_scanner.py -t 192.168.1.1 -p 22,80,443

# Scan a range
python3 port_scanner.py -t 192.168.1.1 -p 1-1024

# Mixed input
python3 port_scanner.py -t 10.0.0.5 -p 22,80,8000-8080

# Only show open ports
python3 port_scanner.py -t 10.0.0.5 -p 1-65535 --open-only

# Adjust timeout and thread count
python3 port_scanner.py -t 192.168.1.1 -p 1-1024 --timeout 0.5 --threads 200
```

### All Flags

| Flag | Default | Description |
|---|---|---|
| `-t`, `--target` | *(prompted)* | Target IP or hostname |
| `-p`, `--ports` | Common ports | Port(s) to scan |
| `--timeout` | `1.0` | Socket timeout in seconds |
| `--threads` | `100` | Max concurrent threads |
| `--open-only` | `False` | Only print open ports |

---

## 📊 Sample Output

```
════════════════════════════════════════════════════════════
  DexAShield Technologies — Port Scanner
════════════════════════════════════════════════════════════
  Target   : 192.168.1.1  (192.168.1.1)
  Ports    : 17 port(s) queued
  Threads  : 100
  Started  : 2025-07-14 10:22:05
────────────────────────────────────────────────────────────

  22      ssh                OPEN  ← SSH-2.0-OpenSSH_8.9p1
  80      http               OPEN
  443     https              OPEN
  3306    mysql              closed
  3389    ms-wbt-server      closed

────────────────────────────────────────────────────────────
  Scan complete in 1.84s
  3 open / 17 scanned

  Open ports summary:
    ▸ 22/ssh — SSH-2.0-OpenSSH_8.9p1
    ▸ 80/http
    ▸ 443/https
════════════════════════════════════════════════════════════
```

---

## 🔬 How It Works

```
Target Input
     │
     ▼
Hostname Resolution (socket.gethostbyname)
     │
     ▼
Port List Parsing (single / range / mixed)
     │
     ▼
ThreadPoolExecutor ──► check_port() per port
                            │
                            ├── TCP connect_ex()
                            ├── IANA service lookup
                            └── Banner grab (recv 1024 bytes)
     │
     ▼
Results Aggregation & Coloured Output
     │
     ▼
Summary Report
```

---

## 🗂️ Default Ports

When no port input is provided, the following common ports are scanned:

| Port | Service |
|---|---|
| 21 | FTP |
| 22 | SSH |
| 23 | Telnet |
| 25 | SMTP |
| 53 | DNS |
| 80 | HTTP |
| 110 | POP3 |
| 143 | IMAP |
| 443 | HTTPS |
| 445 | SMB |
| 3306 | MySQL |
| 3389 | RDP |
| 5900 | VNC |
| 6379 | Redis |
| 8080 | HTTP-alt |
| 8443 | HTTPS-alt |
| 27017 | MongoDB |

---

## ⚠️ Legal Disclaimer

This tool is intended **strictly for authorised security testing and educational purposes only**.

Scanning systems without explicit written permission from the system owner is illegal and may violate:
- The **Nigeria Cybercrimes (Prohibition, Prevention, Etc.) Act, 2015**
- The **Computer Fraud and Abuse Act (CFAA)** and equivalent laws in other jurisdictions

The author assumes no liability for misuse. Always obtain proper authorisation before conducting any security assessment.

Click here to download

[TCP port scanner.py](https://github.com/user-attachments/files/26566767/TCP.port.scanner.py)
