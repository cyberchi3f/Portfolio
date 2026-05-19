# 🖼️ Broken: Gallery — VulnHub Walkthrough

> **Platform:** VulnHub | **URL:** https://www.vulnhub.com/entry/broken-gallery,344/  

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
sudo netdiscover -i eth0
```

**Output:**

<img width="972" height="217" alt="2026-05-19 19_55_01-KALI - VMware Workstation" src="https://github.com/user-attachments/assets/39eb3c68-42e5-4e30-813e-4522543c3351" />


> **Note:** `172.29.209.95` is identified as a VMware guest — confirmed as our target. `172.29.209.47` is our Kali attack machine.

---

## Phase 2 — Service Enumeration

With the target IP confirmed, an aggressive Nmap scan is used to fingerprint services, OS details, and enumerate HTTP content.

```bash
nmap -A 172.29.209.95
```

**Key findings:**

<img width="1080" height="807" alt="2026-05-19 19_56_20-KALI - VMware Workstation" src="https://github.com/user-attachments/assets/330203fd-7861-4372-a319-e1e6e65f4c07" />


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

> ⚠️ **Finding:** Apache is running with directory listing enabled (`-Indexes` not set), exposing all web root files. This is a misconfiguration that aids enumeration significantly.

---

## Phase 3 — Web Enumeration

Navigating to `http://172.29.209.95` in the browser confirms the Apache directory listing with the same files Nmap enumerated.

**Visible files:**

<img width="902" height="485" alt="2026-05-19 19_52_39-KALI - VMware Workstation" src="https://github.com/user-attachments/assets/214058c3-334a-4567-b7cd-fa2736bade27" />

All files are dated **2019-08-09**, providing a timestamp context that later becomes relevant during privilege escalation.

---

## Phase 4 — Steganography Analysis

The presence of multiple image files on a vulnerable machine is a classic indicator of steganography. Investigation follows a structured approach.

### Step 4.1 — Download Images

<img width="797" height="108" alt="image" src="https://github.com/user-attachments/assets/f539836d-7c3b-42c3-90c7-0e0936047465" />

### Step 4.2 — Extract Printable Strings

<img width="557" height="117" alt="image" src="https://github.com/user-attachments/assets/04303cb1-3924-4094-85c4-921bf932c17e" />

The output returns raw JPEG metadata and binary artifact strings — no clear embedded flags or credentials visible in plaintext.

<img width="734" height="1039" alt="2026-05-19 20_01_04-KALI - VMware Workstation" src="https://github.com/user-attachments/assets/5888be69-8ea5-4ec8-acb9-10c09b037e1f" />


### Step 4.3 — Strip Filenames for Reference

<img width="497" height="189" alt="2026-05-19 20_04_25-KALI - VMware Workstation" src="https://github.com/user-attachments/assets/8cad1109-11a3-4fe5-8c1b-1b311bc79d7b" />

```
ls |grep .jpg | sed s/\.jpg// > wordlists2
```

This command lists all files in the current directory, filters for files ending in `.jpg`, removes the `.jpg` extension from each filename, and saves the resulting names to a file called `wordlists2`. These filenames can then be used as potential passwords or keywords for further analysis.

### Step 4.4 — Steghide Extraction Attempts

<img width="901" height="517" alt="2026-05-19 20_14_05-KALI - VMware Workstation" src="https://github.com/user-attachments/assets/8f7f6fb1-b496-455f-91ce-2289a375c139" />

**Result:** I used the filename without the file extension as the passphrase during the extraction process. All four return: `steghide: could not extract any data with that passphrase!`

> 💡 **Pivot point:** Steganography with a known passphrase might succeed, but for now we have no passphrase. The filenames themselves become our next wordlist seed — a CTF methodology pivot.

---

## Phase 5 — Wordlist Crafting

Rather than using generic wordlists like `rockyou.txt`, the attack surface itself — the page names and image filenames — is used to build a targeted wordlist.

### Step 5.1 — Build Initial Wordlist from Context

The filenames and page names from the web server provide natural candidates:

<img width="666" height="159" alt="2026-05-19 20_21_28-KALI - VMware Workstation" src="https://github.com/user-attachments/assets/1361d436-39d6-43e2-a926-3a445f8641c3" />

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

<img width="1581" height="202" alt="2026-05-19 20_51_03-KALI - VMware Workstation" src="https://github.com/user-attachments/assets/de56fcff-4d1d-41f3-8930-206c46c0ac04" />

This generates the `mangled` file — a large mutated password dictionary built from context-aware seed words.

---

## Phase 6 — SSH Brute Force

### Attempt 1 — Using Original Wordlist (Failed)

<img width="1896" height="298" alt="2026-05-19 20_17_04-KALI - VMware Workstation" src="https://github.com/user-attachments/assets/5d161d1b-f879-4713-bb1a-d031271f490c" />

**Result:** `1 of 1 target completed, 0 valid password found`  
16 login attempts across 4 usernames × 4 passwords — no hits.

### Attempt 2 — Using Crafted Wordlist3 (Success ✅)

<img width="1907" height="321" alt="2026-05-19 20_52_15-KALI - VMware Workstation" src="https://github.com/user-attachments/assets/17bb3e68-847b-4344-bccf-5f075bd90f4f" />

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

<img width="1159" height="598" alt="2026-05-19 20_56_47-KALI - VMware Workstation" src="https://github.com/user-attachments/assets/43c8a273-c6c1-42d6-8a8c-97467a7e161e" />

SSH prompts for host key verification (first connection) — accepted. Password entered: `broken`

> ⚠️ **Observation:** 762 pending package updates including 458 security patches indicates a severely outdated and unpatched system — a significant attack surface in a real-world scenario.

---

## Phase 8 — Post-Exploitation Enumeration

### Step 8.1 — Directory Listing

<img width="1202" height="877" alt="2026-05-19 21_02_11-KALI - VMware Workstation" src="https://github.com/user-attachments/assets/c2ef3a49-4193-41d5-98e0-a43af5a31263" />

Notable files in `/home/broken`:

```
-rw——————   .bash_history  
-rw-r--r--  .sudo_as_admin_successful  
drwxr-xr-x  Pictures/
```

> 🔑 **Key finding:** `.sudo_as_admin_successful` exists — this file is created by `sudo` when a user in the `sudo` group successfully runs a command. It confirms `broken` is in the `sudo` group.

### Step 8.2 — Bash History Review

<img width="997" height="1020" alt="2026-05-19 20_57_46-KALI - VMware Workstation" src="https://github.com/user-attachments/assets/e69ca957-5e92-4a59-9d8c-3b434d47c2e5" />

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

<img width="853" height="600" alt="2026-05-19 21_03_43-KALI - VMware Workstation" src="https://github.com/user-attachments/assets/6bc5427b-42f3-4e8a-851a-9ca1e43e27f0" />


> 🔑 **Vulnerability identified:** This init script runs as root at boot time (via `/etc/init.d/`). If the system day is **Thursday (day 4)**, it sets the root password to `TodayIsAgoodDay`.

### Step 8.4 — Verify Current Day

<img width="1424" height="252" alt="2026-05-19 21_06_09-KALI - VMware Workstation" src="https://github.com/user-attachments/assets/d7ba3278-f953-4b61-99b1-defb4e0bf282" />

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

<img width="1048" height="391" alt="image" src="https://github.com/user-attachments/assets/f27f83ee-ee2d-49a1-b0ed-0803f2077db4" />

Result: Connection closed — reboot confirmed. This validates that `broken` can reboot the machine via sudo.

### Step 9.2 — Attempt Direct Script Execution (Blocked)

<img width="1363" height="87" alt="image" src="https://github.com/user-attachments/assets/023e9d68-856f-466d-83b0-b2628e5a858d" />

Direct execution is blocked by sudoers policy — the init.d mechanism is the intended trigger.

### Step 9.3 — Set Date to Thursday and Reboot

2026-05-14 was a Thursday (day 4)

<img width="851" height="114" alt="image" src="https://github.com/user-attachments/assets/1ba3efac-2ca4-4793-93e7-a9d5e1dafd42" />


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
<img width="986" height="581" alt="image" src="https://github.com/user-attachments/assets/371fe41c-0566-453d-85a0-906ac666caf7" />

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
