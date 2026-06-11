<div align="center">

# 🕵️‍♂️ Steganography Lab
### Concealment & Detection Techniques on Kali Linux


## 📌 Overview

This lab is a **hands-on walkthrough of steganography techniques** using `steghide` on Kali Linux. It covers the complete lifecycle — concealment, forensic analysis, cryptographic verification, extraction, and attacker OPSEC cleanup — across both **audio** and **image** carrier formats.

> **Steganography** hides the *existence* of secret data inside innocent-looking files.  
> **Cryptography** hides the *content*. When combined — as in this lab — the result is a layered covert channel: encrypted *and* invisible.


---

#<img width="449" height="65" alt="Lab Files" src="https://github.com/user-attachments/assets/7c75abc7-eb72-42cd-9396-d0556b909322" />
---

## 🧰 Environment & Tools

| Component | Details |
|---|---|
| **OS** | Kali Linux (VMware Workstation) |
| **Steganography** | `steghide` |
| **Forensic Analysis** | `exiftool`, `stat`, `file`, `sha256sum` |
| **Carrier Formats** | JPEG (`.jpg`), WAV PCM Audio (`.wav`) |
| **Encryption Ciphers** | Rijndael-128/CBC (default), Blowfish/CBC (`-e blowfish`) |
| **Working Directory** | `~/LAB - steghide` |

---

## 📂 Lab File Structure

<img width="449" height="65" alt="Lab Files" src="https://github.com/user-attachments/assets/9a339f99-570d-4702-a4aa-2b7f7d1a4dd8" />

---

## 🔬 Lab Walkthrough

###  01 — Carrier Verification

Before any operations, `file` reads the magic bytes of each carrier to confirm its true format — a critical forensic baseline.

<img width="1139" height="151" alt="File Properties" src="https://github.com/user-attachments/assets/d029335e-efb1-40eb-a477-b03e913fdc9d" />

> ✅ Both carriers confirmed valid. Both are natively supported Steghide formats.

---

### 02 — Embed Shell Script in WAV (Rijndael-128)

Using Steghide's **default Rijndael-128 CBC** encryption with zlib compression:


<img width="399" height="99" alt="2026-06-11 05_53_11-KALI - VMware Workstation" src="https://github.com/user-attachments/assets/7680c666-7263-4353-b048-9954978be687" />

**Verify the embedding:**

<img width="399" height="99" alt="2026-06-11 05_53_11-KALI - VMware Workstation" src="https://github.com/user-attachments/assets/d6f8c416-4769-4bce-9e99-9ddc56549cd4" />



| Field | Value |
|---|---|
| Embedded File | `hello.sh` |
| Payload Size | 31 Bytes |
| Encryption | Rijndael-128, CBC |
| Compressed | Yes (zlib) |
| Carrier Capacity | 646.0 KB |

> 📝 The WAV file is modified in-place at the binary level — but to any media player or listener, it sounds **completely identical**. The 646 KB capacity dwarfs the 31-byte payload.

---

### § 03 — Embed Text File in JPEG (Blowfish)

Explicitly specifying **Blowfish CBC** to demonstrate cipher flexibility:

<img width="399" height="99" alt="2026-06-11 05_53_11-KALI - VMware Workstation" src="https://github.com/user-attachments/assets/860f4d71-36e4-46c0-892c-46954d01e5be" />

**Verify the embedding:**

<img width="399" height="99" alt="2026-06-11 05_53_11-KALI - VMware Workstation" src="https://github.com/user-attachments/assets/f89b195c-9575-42df-9678-4b57da4c1223" />

| Field | Value |
|---|---|
| Embedded File | `secret.txt` |
| Payload Size | 19 Bytes |
| Encryption | Blowfish, CBC |
| Compressed | Yes |
| Carrier Capacity | **10.8 KB** *(limited)* |

> ⚠️ JPEG carriers have **significantly lower embedding capacity** than WAV files. Adequate for short text payloads; insufficient for binaries or large files.

---

### 04 — Metadata Forensics

#### 4a. ExifTool — Header Analysis

<img width="399" height="99" alt="2026-06-11 05_53_11-KALI - VMware Workstation" src="https://github.com/user-attachments/assets/81e564e6-c0f2-48f4-8f87-9f1bccdfc31c" />


> 🔎 **No EXIF anomalies detected.** Steghide modifies **pixel-level LSBs** in the image data section — not the EXIF header. An analyst relying solely on `exiftool` would find nothing suspicious.

#### 4b. Filesystem Timestamps — `stat`

<img width="399" height="99" alt="2026-06-11 05_53_11-KALI - VMware Workstation" src="https://github.com/user-attachments/assets/1fd922b1-62bd-4d68-a945-6793b4106ef3" />


> 🔎 The **~20-minute delta** between `Birth` and `Modify` timestamps is consistent with the embedding operation occurring post-creation. In a real investigation, this timestamp divergence is a forensic lead — though it can be forged with `touch` and must be corroborated.

---

### § 05 — Payload Extraction & Execution

#### Extract from WAV

<img width="360" height="79" alt="extract steghide script" src="https://github.com/user-attachments/assets/858191bf-e431-4bc8-86bc-b396796df405" />

#### Execute the Extracted Script

<img width="359" height="112" alt="extracted script" src="https://github.com/user-attachments/assets/0a72ce47-eefe-4aa8-a5c6-c0e60f6bd7dd" />

#### Extract from JPEG
<img width="346" height="80" alt="extract steghide txt file" src="https://github.com/user-attachments/assets/fea8ce94-8d5b-42a7-9b21-64e8fc898626" />

<img width="346" height="80" alt="extract steghide txt file" src="https://github.com/user-attachments/assets/770ce5b9-fe66-4d9a-9f89-b5b0c038f1ac" />

> 🔴 **Attack implication:** A threat actor could embed a malicious shell script inside an innocuous WAV audio file, distribute it via email or messaging, and have the recipient unknowingly extract and execute it — **bypassing AV tools and content inspection systems** that only analyse file headers or extensions.

---

### § 06 — SHA-256 Integrity Verification

Comparing the stego-modified `cover.jpg` against a clean baseline `main-cover.jpg`:

<img width="346" height="80" alt="extract steghide txt file" src="https://github.com/user-attachments/assets/e1a46339-6389-46f9-a643-f789cc2f93dd" />


| File | SHA-256 Hash | Status |
|---|---|---|
| `cover.jpg` *(stego)* | `e82d7128...c18dfb` | 🔴 Modified |
| `main-cover.jpg` *(clean)* | `3b5f33e2...b1af0` | 🟢 Original |

> 🔑 **Hash mismatch conclusively proves file modification** — even though the images appear visually identical. Any FIM/HIDS system with hash-based detection would immediately flag this divergence.

---

### § 07 — OPSEC Cleanup

<img width="361" height="114" alt="Deleted embedded files" src="https://github.com/user-attachments/assets/7c8948f6-d9ca-42d8-93d9-c9c640061b96" />


> Only media files remain. The hidden payloads are **still fully preserved** inside the carriers and extractable with the correct passphrase — demonstrating that cleanup leaves no visible trace of the operation.

---

## 🔴 Red Team Insights

- WAV audio provides **20× more embedding capacity** than JPEG for the same file size
- Embedding does **not alter EXIF metadata** — bypasses header-only forensic tools
- Combined encryption + steganography = **dual-layer covert channel**: encrypted AND invisible
- Passphrase protection means detected stego objects **cannot be read** without the key
- Post-operation cleanup leaves the filesystem looking entirely normal

---

## 🔵 Blue Team Countermeasures

- **File Integrity Monitoring (FIM):** Maintain SHA-256 baselines for all media assets — hash mismatch = immediate flag
- **Steganalysis Tools:** `zsteg`, `StegExpose`, `stegdetect` — deploy for proactive scanning
- **Statistical Analysis:** Chi-square test on image pixel histograms detects LSB embedding anomalies
- **DLP Policies:** Flag and inspect media files (`.wav`, `.jpg`) transmitted via email or USB
- **Timestamp Correlation:** Unusual `Birth` → `Modify` deltas on media files warrant investigation
- **Deep Content Inspection:** Inspect binary structure, not just MIME type or extension

---

## 🕳️ Detection Gaps

| Gap | Detail |
|---|---|
| `exiftool` alone insufficient | Steghide modifies image data, not the EXIF header |
| Timestamp delta is a weak signal | Can be forged with `touch`; requires corroboration |
| Visual inspection useless | Stego carriers are perceptually identical to originals |
| AV / signature tools blind | No malware signature exists for passphrase-encrypted payloads |
| **Reliable detection path** | Hash baseline comparison + dedicated steganalysis tools |

---


## 🧠 Skills Demonstrated

`Steganography` · `Digital Forensics` · `File Integrity Monitoring` · `Cryptography`  
`Metadata Analysis` · `Steganalysis` · `Kali Linux` · `Bash Scripting`  
`Incident Response` · `Threat Intelligence` · `OPSEC` · `Red Team Techniques`


## ⚖️ Disclaimer

> This lab was conducted in a **controlled, isolated virtual environment** for educational and portfolio purposes only. All techniques demonstrated are intended strictly for **authorised security testing, research, and awareness training**. Unauthorised use of steganography tools against systems or individuals without explicit permission is illegal and unethical.

---
