<div align="center">

# 🕵️‍♂️ Steganography Lab
### Concealment & Detection Techniques on Kali Linux

*"Break it ethically. Fix it permanently."* — **cyberchi3f**

---

![Platform](https://img.shields.io/badge/Platform-Kali%20Linux-557C94?style=for-the-badge&logo=kalilinux&logoColor=white)
![Tool](https://img.shields.io/badge/Tool-Steghide-1A4A7A?style=for-the-badge&logo=linux&logoColor=white)
![Domain](https://img.shields.io/badge/Domain-Digital%20Forensics-6B2FA0?style=for-the-badge&logo=hackthebox&logoColor=white)
![Ref](https://img.shields.io/badge/Ref-LAB--STEG--2026--001-B45309?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Completed-2E7D32?style=for-the-badge&logo=checkmarx&logoColor=white)

</div>

---

## 📌 Overview

This lab is a **hands-on walkthrough of steganography techniques** using `steghide` on Kali Linux. It covers the complete lifecycle — concealment, forensic analysis, cryptographic verification, extraction, and attacker OPSEC cleanup — across both **audio** and **image** carrier formats.

> **Steganography** hides the *existence* of secret data inside innocent-looking files.  
> **Cryptography** hides the *content*. When combined — as in this lab — the result is a layered covert channel: encrypted *and* invisible.

This walkthrough is documented as part of the **cyberchi3f** offensive & forensics portfolio under [DexAShield Technologies](https://github.com/cyberchi3f).

---

## 🎯 Objectives

- [x] Verify carrier file formats using magic-byte analysis (`file`)
- [x] Embed an executable shell script inside a **WAV audio** file (Rijndael-128/CBC)
- [x] Embed a text file inside a **JPEG image** using **Blowfish/CBC** encryption
- [x] Confirm embedded payloads via `steghide info`
- [x] Perform metadata forensics with `exiftool` and `stat`
- [x] Extract and execute hidden payloads end-to-end
- [x] Prove file tampering via **SHA-256 hash comparison**
- [x] Simulate attacker **OPSEC cleanup** procedures

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

```
LAB - steghide/
├── audio.wav          # WAV carrier  — 646 KB, PCM 16-bit stereo 44100 Hz
├── cover.jpg          # JPEG carrier — 186 KB, 959×1480, JFIF 1.01
├── cover2.jpg         # Alternate image carrier
├── hello.sh           # Payload A — shell script  → embedded in audio.wav
└── secret.txt         # Payload B — text file     → embedded in cover.jpg
```

---

## 🔬 Lab Walkthrough

### § 01 — Carrier Verification

Before any operations, `file` reads the magic bytes of each carrier to confirm its true format — a critical forensic baseline.

```bash
$ file cover.jpg
cover.jpg: JPEG image data, JFIF standard 1.01, aspect ratio, density 1×1,
           segment length 16, progressive, precision 8, 959×1480, components 3

$ file audio.wav
audio.wav: RIFF (little-endian) data, WAVE audio, Microsoft PCM, 16 bit, stereo 44100 Hz
```

> ✅ Both carriers confirmed valid. Both are natively supported Steghide formats.

---

### § 02 — Embed Shell Script in WAV (Rijndael-128)

Using Steghide's **default Rijndael-128 CBC** encryption with zlib compression:

```bash
$ steghide embed -cf audio.wav -ef hello.sh
Enter passphrase:
Re-Enter passphrase:
embedding "hello.sh" in "audio.wav"... done
```

**Verify the embedding:**

```bash
$ steghide info audio.wav
"audio.wav":
  format: wave audio, PCM encoding
  capacity: 646.0 KB
Try to get information about embedded data ? (y/n) y
Enter passphrase:
  embedded file "hello.sh":
    size: 31.0 Byte
    encrypted: rijndael-128, cbc
    compressed: yes
```

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

```bash
$ steghide embed -cf cover.jpg -ef secret.txt -e blowfish
Enter passphrase:
Re-Enter passphrase:
embedding "secret.txt" in "cover.jpg"... done
```

**Verify the embedding:**

```bash
$ steghide info cover.jpg
"cover.jpg":
  format: jpeg
  capacity: 10.8 KB
Try to get information about embedded data ? (y/n) y
Enter passphrase:
  embedded file "secret.txt":
    size: 19.0 Byte
    encrypted: blowfish, cbc
    compressed: yes
```

| Field | Value |
|---|---|
| Embedded File | `secret.txt` |
| Payload Size | 19 Bytes |
| Encryption | Blowfish, CBC |
| Compressed | Yes |
| Carrier Capacity | **10.8 KB** *(limited)* |

> ⚠️ JPEG carriers have **significantly lower embedding capacity** than WAV files. Adequate for short text payloads; insufficient for binaries or large files.

---

### § 04 — Metadata Forensics

#### 4a. ExifTool — Header Analysis

```bash
$ exiftool cover.jpg
ExifTool Version Number  : 13.44
File Name                : cover.jpg
File Size                : 186 kB
File Modification Date   : 2026:06:10 23:39:02-05:00
File Type                : JPEG
MIME Type                : image/jpeg
JFIF Version             : 1.01
Image Width              : 959
Image Height             : 1480
Encoding Process         : Baseline DCT, Huffman coding
Bits Per Sample          : 8
Color Components         : 3
Megapixels               : 1.4
```

> 🔎 **No EXIF anomalies detected.** Steghide modifies **pixel-level LSBs** in the image data section — not the EXIF header. An analyst relying solely on `exiftool` would find nothing suspicious.

#### 4b. Filesystem Timestamps — `stat`

```bash
$ stat cover.jpg
  File: cover.jpg
  Size: 186169    Blocks: 368    IO Block: 4096   regular file
Device: 8,1       Inode: 1116675   Links: 1
Access: (0664/-rw-rw-r--)  Uid: (1000/cyberchief)  Gid: (1000/cyberchief)
Birth:  2026-06-10 23:19:09.911932342 -0500
Modify: 2026-06-10 23:39:02.977228278 -0500   ← ~20 min after birth
Change: 2026-06-10 23:39:02.977228278 -0500
```

> 🔎 The **~20-minute delta** between `Birth` and `Modify` timestamps is consistent with the embedding operation occurring post-creation. In a real investigation, this timestamp divergence is a forensic lead — though it can be forged with `touch` and must be corroborated.

---

### § 05 — Payload Extraction & Execution

#### Extract from WAV

```bash
$ steghide extract -sf audio.wav
Enter passphrase:
wrote extracted data to "hello.sh".
```

#### Execute the Extracted Script

```bash
$ chmod +x ./hello.sh
$ ./hello.sh
Hello World
```

#### Extract from JPEG

```bash
$ steghide extract -sf cover.jpg
Enter passphrase:
wrote extracted data to "secret.txt".

$ cat secret.txt
Confidential Notes
```

> 🔴 **Attack implication:** A threat actor could embed a malicious shell script inside an innocuous WAV audio file, distribute it via email or messaging, and have the recipient unknowingly extract and execute it — **bypassing AV tools and content inspection systems** that only analyse file headers or extensions.

---

### § 06 — SHA-256 Integrity Verification

Comparing the stego-modified `cover.jpg` against a clean baseline `main-cover.jpg`:

```bash
$ sha256sum cover.jpg
e82d7128949c23e6d9b6b8aa76eb041fa3d70712d18112d790b90cd51ec18dfb  cover.jpg

$ sha256sum main-cover.jpg
3b5f33e27df8605e801c4d000de9936b83a06cc0b9bd629de87244ad050b1af0  main-cover.jpg
```

| File | SHA-256 Hash | Status |
|---|---|---|
| `cover.jpg` *(stego)* | `e82d7128...c18dfb` | 🔴 Modified |
| `main-cover.jpg` *(clean)* | `3b5f33e2...b1af0` | 🟢 Original |

> 🔑 **Hash mismatch conclusively proves file modification** — even though the images appear visually identical. Any FIM/HIDS system with hash-based detection would immediately flag this divergence.

---

### § 07 — OPSEC Cleanup

```bash
$ rm hello.sh secret.txt

$ ls
audio.wav  cover2.jpg  cover.jpg
```

> Only media files remain. The hidden payloads are **still fully preserved** inside the carriers and extractable with the correct passphrase — demonstrating that cleanup leaves no visible trace of the operation.

---

## 📊 Operations Summary

| # | Operation | Command | Carrier | Outcome |
|---|---|---|---|---|
| 01 | File Survey | `ls` | All | Files confirmed |
| 02 | Carrier Verification | `file` | JPG / WAV | Formats validated |
| 03 | Embed Shell Script | `steghide embed` | `audio.wav` | `hello.sh` hidden — Rijndael-128 |
| 04 | Verify Audio Embed | `steghide info` | `audio.wav` | 31B · encrypted · compressed |
| 05 | Embed Text File | `steghide embed -e blowfish` | `cover.jpg` | `secret.txt` hidden — Blowfish |
| 06 | Verify Image Embed | `steghide info` | `cover.jpg` | 19B · encrypted · compressed |
| 07 | EXIF Analysis | `exiftool` | `cover.jpg` | No EXIF anomalies found |
| 08 | Filesystem Timestamps | `stat` | `cover.jpg` | 20-min Birth→Modify delta flagged |
| 09 | Extract from WAV | `steghide extract -sf` | `audio.wav` | `hello.sh` recovered ✅ |
| 10 | Execute Payload | `chmod +x` / `./hello.sh` | — | Output: `Hello World` ✅ |
| 11 | Extract from JPEG | `steghide extract -sf` | `cover.jpg` | `secret.txt` recovered ✅ |
| 12 | Hash Comparison | `sha256sum` | `cover.jpg` | Hash mismatch — tampering proven |
| 13 | OPSEC Cleanup | `rm` | — | Payload originals deleted |

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

## 📁 Repository Contents

```
📦 steganography-lab/
├── 📄 README.md                                  ← You are here
├── 📝 Steganography_Lab_Report_Simon_Etim.docx   ← Full technical report (Word)
└── 🌐 steganography_lab_report.html              ← Interactive HTML report
```

---

## 🧠 Skills Demonstrated

`Steganography` · `Digital Forensics` · `File Integrity Monitoring` · `Cryptography`  
`Metadata Analysis` · `Steganalysis` · `Kali Linux` · `Bash Scripting`  
`Incident Response` · `Threat Intelligence` · `OPSEC` · `Red Team Techniques`

---

## 👤 Author

<div align="center">

**Simon Etim** — *cyberchi3f*

VAPT Specialist · Cybersecurity Instructor · Founder, DexAShield Technologies

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Simon%20Etim-0077B5?style=flat-square&logo=linkedin&logoColor=white)](https://linkedin.com/in/simon-etim)
[![GitHub](https://img.shields.io/badge/GitHub-cyberchi3f-181717?style=flat-square&logo=github&logoColor=white)](https://github.com/cyberchi3f)
[![Email](https://img.shields.io/badge/Email-DexAShield-D14836?style=flat-square&logo=gmail&logoColor=white)](mailto:info@dexashield.com)

*"Driven by Technology, Defined by Security"*

</div>

---

## ⚖️ Disclaimer

> This lab was conducted in a **controlled, isolated virtual environment** for educational and portfolio purposes only. All techniques demonstrated are intended strictly for **authorised security testing, research, and awareness training**. Unauthorised use of steganography tools against systems or individuals without explicit permission is illegal and unethical.

---

<div align="center">
<sub>© 2026 DexAShield Technologies · LAB-STEG-2026-001</sub>
</div>
