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


[TCP port scanner.py](https://github.com/user-attachments/files/26566767/TCP.port.scanner.py)
#!/usr/bin/env python3
"""
port_scanner.py — Simple TCP Port Scanner
DexAShield Technologies | Simon Etim
Usage: python3 port_scanner.py
"""

import socket
import concurrent.futures
import argparse
import sys
from datetime import datetime


# ─── ANSI colours (graceful fallback on Windows) ────────────────────────────
try:
    import colorama
    colorama.init(autoreset=True)
    GREEN  = "\033[92m"
    RED    = "\033[91m"
    YELLOW = "\033[93m"
    CYAN   = "\033[96m"
    BOLD   = "\033[1m"
    RESET  = "\033[0m"
except ImportError:
    GREEN = RED = YELLOW = CYAN = BOLD = RESET = ""


# ─── Port scanner core ───────────────────────────────────────────────────────

def check_port(host: str, port: int, timeout: float = 1.0) -> dict:
    """
    Attempt a TCP connection to host:port.
    Returns a dict with status, banner (if any), and service guess.
    """
    result = {"port": port, "status": "closed", "service": "", "banner": ""}

    # Guess service name from IANA registry
    try:
        result["service"] = socket.getservbyport(port, "tcp")
    except OSError:
        result["service"] = "unknown"

    try:
        with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as sock:
            sock.settimeout(timeout)
            conn = sock.connect_ex((host, port))
            if conn == 0:
                result["status"] = "open"
                # Grab banner (best-effort)
                try:
                    sock.settimeout(1.0)
                    banner_raw = sock.recv(1024)
                    result["banner"] = banner_raw.decode(errors="replace").strip()
                except Exception:
                    pass
    except socket.gaierror:
        result["status"] = "error"
        result["service"] = "hostname resolution failed"
    except Exception as e:
        result["status"] = "error"
        result["service"] = str(e)

    return result


def resolve_host(host: str) -> str:
    """Resolve hostname to IP for display purposes."""
    try:
        return socket.gethostbyname(host)
    except socket.gaierror:
        return "unresolved"


def parse_ports(port_input: str) -> list[int]:
    """
    Parse a flexible port specification into a sorted list of ints.
    Accepts: 80,443,8080  |  1-1024  |  22,80,8000-8080
    """
    ports = set()
    for part in port_input.split(","):
        part = part.strip()
        if "-" in part:
            start, end = part.split("-", 1)
            ports.update(range(int(start), int(end) + 1))
        else:
            ports.add(int(part))
    return sorted(ports)


# ─── Output helpers ──────────────────────────────────────────────────────────

def print_banner(host: str, ip: str, ports: list[int], threads: int):
    print(f"\n{CYAN}{BOLD}{'═' * 60}{RESET}")
    print(f"{CYAN}{BOLD}  DexAShield Technologies — Port Scanner{RESET}")
    print(f"{CYAN}{BOLD}{'═' * 60}{RESET}")
    print(f"  Target   : {BOLD}{host}{RESET}  ({ip})")
    print(f"  Ports    : {len(ports)} port(s) queued")
    print(f"  Threads  : {threads}")
    print(f"  Started  : {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")
    print(f"{CYAN}{'─' * 60}{RESET}\n")


def print_result(r: dict):
    port_col = f"{r['port']:<6}"
    svc_col  = f"{r['service']:<18}"
    if r["status"] == "open":
        status_col = f"{GREEN}OPEN{RESET}"
    elif r["status"] == "closed":
        status_col = f"{RED}closed{RESET}"
    else:
        status_col = f"{YELLOW}{r['status']}{RESET}"

    line = f"  {BOLD}{port_col}{RESET}  {svc_col}  {status_col}"
    if r["banner"]:
        line += f"  ← {YELLOW}{r['banner'][:60]}{RESET}"
    print(line)


def print_summary(results: list[dict], elapsed: float):
    open_ports = [r for r in results if r["status"] == "open"]
    print(f"\n{CYAN}{'─' * 60}{RESET}")
    print(f"  Scan complete in {elapsed:.2f}s")
    print(f"  {GREEN}{BOLD}{len(open_ports)} open{RESET} / {len(results)} scanned\n")
    if open_ports:
        print(f"  {BOLD}Open ports summary:{RESET}")
        for r in open_ports:
            banner_note = f" — {r['banner'][:50]}" if r["banner"] else ""
            print(f"    {GREEN}▸{RESET} {r['port']}/{r['service']}{banner_note}")
    print(f"{CYAN}{'═' * 60}{RESET}\n")


# ─── Main ────────────────────────────────────────────────────────────────────

def main():
    parser = argparse.ArgumentParser(
        description="Simple TCP Port Scanner — DexAShield Technologies",
        formatter_class=argparse.RawTextHelpFormatter,
    )
    parser.add_argument(
        "-t", "--target",
        default=None,
        help="Target host (IP or hostname). Prompted if omitted.",
    )
    parser.add_argument(
        "-p", "--ports",
        default=None,
        help=(
            "Port(s) to scan.\n"
            "  Examples: 80            → single port\n"
            "            22,80,443     → comma-separated\n"
            "            1-1024        → range\n"
            "            22,80,8000-9000\n"
            "  Default : common ports (21,22,23,25,53,80,110,143,443,\n"
            "            445,3306,3389,5900,6379,8080,8443,27017)"
        ),
    )
    parser.add_argument(
        "--timeout",
        type=float,
        default=1.0,
        help="Socket timeout in seconds (default: 1.0)",
    )
    parser.add_argument(
        "--threads",
        type=int,
        default=100,
        help="Max concurrent threads (default: 100)",
    )
    parser.add_argument(
        "--open-only",
        action="store_true",
        help="Only print open ports",
    )
    args = parser.parse_args()

    # ── Interactive prompts if flags omitted ─────────────────────────────────
    target = args.target or input("  Enter target host/IP: ").strip()
    if not target:
        print(f"{RED}  No target specified. Exiting.{RESET}")
        sys.exit(1)

    DEFAULT_PORTS = "21,22,23,25,53,80,110,143,443,445,3306,3389,5900,6379,8080,8443,27017"

    if args.ports is None:
        port_input = input(
            f"  Ports to scan [{DEFAULT_PORTS[:40]}...] (Enter for defaults): "
        ).strip()
        port_input = port_input or DEFAULT_PORTS
    else:
        port_input = args.ports

    try:
        ports = parse_ports(port_input)
    except ValueError as e:
        print(f"{RED}  Invalid port specification: {e}{RESET}")
        sys.exit(1)

    ip = resolve_host(target)
    print_banner(target, ip, ports, args.threads)

    # ── Scan ─────────────────────────────────────────────────────────────────
    start = datetime.now()
    results = []

    with concurrent.futures.ThreadPoolExecutor(max_workers=args.threads) as executor:
        futures = {
            executor.submit(check_port, target, p, args.timeout): p
            for p in ports
        }
        for future in concurrent.futures.as_completed(futures):
            r = future.result()
            results.append(r)
            if args.open_only:
                if r["status"] == "open":
                    print_result(r)
            else:
                print_result(r)

    elapsed = (datetime.now() - start).total_seconds()

    # Sort results by port for the summary
    results.sort(key=lambda x: x["port"])
    print_summary(results, elapsed)


if __name__ == "__main__":
    main()



