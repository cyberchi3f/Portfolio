# 🕵️ Steganography — Concealment & Detection Techniques

> **Lab Objective:** Demonstrate the full steganography lifecycle using `steghide` on Kali Linux, including data concealment inside audio and image carrier files, metadata forensics, cryptographic integrity verification, payload extraction and execution, and attacker OPSEC cleanup simulation.

---

## 📋 Table of Contents

- [Lab Environment](#lab-environment)
- [Background](#background)
- [Lab Walkthrough](#lab-walkthrough)
  * [1. Working Directory Survey](#1-working-directory-survey)
  * [2. Carrier File Type Verification](#2-carrier-file-type-verification)
  * [3. Embed Shell Script in WAV Audio File](#3-embed-shell-script-in-wav-audio-file)
  * [4. Verify Audio Embedding](#4-verify-audio-embedding)
  * [5. Embed Text File in JPEG Image (Blowfish)](#5-embed-text-file-in-jpeg-image-blowfish)
  * [6. Verify Image Embedding](#6-verify-image-embedding)
  * [7. Metadata Forensics — ExifTool Analysis](#7-metadata-forensics--exiftool-analysis)
  * [8. Filesystem Timestamps — stat Command](#8-filesystem-timestamps--stat-command)
  * [9. Extract Shell Script from WAV](#9-extract-shell-script-from-wav)
  * [10. Execute the Extracted Payload](#10-execute-the-extracted-payload)
  * [11. Extract Text File from JPEG](#11-extract-text-file-from-jpeg)
  * [12. SHA-256 Integrity Verification](#12-sha-256-integrity-verification)
  * [13. OPSEC Cleanup](#13-opsec-cleanup)
- [Steganography vs Cryptography](#steganography-vs-cryptography)
- [Detection Gaps & Countermeasures](#detection-gaps--countermeasures)
- [Verification Summary](#verification-summary)
- [Result](#result)
- [Conclusion](#conclusion)
- [Tags](#tags)

---

## 🖥️ Lab Environment

| Parameter | Value |
|---|---|
| **Operating System** | Kali Linux (VMware Workstation) |
| **Primary Tool** | `steghide` |
| **Analysis Tools** | `exiftool`, `stat`, `file`, `sha256sum` |
| **Carrier Formats** | JPEG (`.jpg`), WAV PCM Audio (`.wav`) |
| **Encryption Ciphers** | Rijndael-128/CBC (default), Blowfish/CBC (`-e blowfish`) |


---

## 📖 Background

Steganography is the practice of **concealing a secret message, file, or payload inside an ordinary-looking carrier object** without altering its apparent form or function. Unlike cryptography — which protects the *content* of data — steganography hides the very *existence* of the data.

When combined, the two techniques produce a layered covert communication channel:

- **Cryptography** encrypts the payload so it cannot be read
- **Steganography** hides the encrypted payload inside a carrier so it cannot be found

`steghide` is an open-source steganography utility that supports JPEG, BMP, WAV, and AU carriers. It encrypts the payload before embedding, meaning that even if an investigator detects the presence of hidden data, they cannot extract it without the correct passphrase.

## ⚙️ Lab Walkthrough

### 1. Working Directory Survey

Before any operations begin, the working directory is listed to confirm all carrier files and payloads are present.

<img width="449" height="65" alt="Lab Files" src="https://github.com/user-attachments/assets/98ccebf5-5c7f-4288-9992-166bb55c5b4a" />

> **Figure 2:** Lab directory contents confirmed — carriers and payloads ready

---

### 2. Carrier File Type Verification

The `file` command reads the **magic bytes** of each carrier to confirm its true format. This is a critical forensic baseline before any embedding is performed.

<img width="361" height="114" alt="Deleted embedded files" src="https://github.com/user-attachments/assets/8721d2cb-5061-498f-9aec-df4b22b3f077" />

> **Figure 3:** Both carrier files confirmed valid — JPEG 959×1480 and WAV PCM 44100 Hz

---

### 3. Embed Shell Script in WAV Audio File

The `steghide embed` command hides `hello.sh` inside `audio.wav` using **Rijndael-128 CBC** encryption (the default cipher) with zlib compression. The `-cf` flag specifies the cover file; the `-ef` flag specifies the file to embed.

<img width="360" height="79" alt="extract steghide script" src="https://github.com/user-attachments/assets/298b917b-936d-49a0-9032-b556caa52469" />

> **Figure 4:** Shell script successfully embedded in WAV audio carrier

### 4. Verify Audio Embedding

`steghide info` confirms whether embedded data exists in the carrier. When the correct passphrase is supplied, it reveals the embedded filename, payload size, encryption algorithm, and compression status.

<img width="360" height="79" alt="extract steghide script" src="https://github.com/user-attachments/assets/19566663-77a5-47b0-9cab-357b9ef326e7" />

> **Figure 5:** Embedding verified — 31-byte payload, Rijndael-128 CBC, compressed

> 📝 **Note:** The WAV file is modified in-place at the binary level but remains perceptually identical to any media player. The 646 KB carrier capacity far exceeds the 31-byte payload.



### 5. Embed Text File in JPEG Image (Blowfish)

The `-e blowfish` flag explicitly selects **Blowfish CBC** as the encryption cipher, overriding the Rijndael default. This demonstrates that `steghide` supports configurable cipher selection per embedding operation.

<img width="360" height="79" alt="extract steghide script" src="https://github.com/user-attachments/assets/92eb02e2-ea2a-4d34-9f42-024cf5012516" />

> **Figure 6:** Text file successfully embedded in JPEG carrier using Blowfish encryption


### 6. Verify Image Embedding

Running `steghide info` on the modified JPEG confirms the Blowfish cipher is active and reveals the embedded payload details.

<img width="360" height="79" alt="extract steghide script" src="https://github.com/user-attachments/assets/071e7401-2ddb-4cbe-aa2b-2ffc6d53dba6" />

> **Figure 7:** Embedding verified — 19-byte payload, Blowfish CBC, compressed

> ⚠️ **Note:** JPEG carriers have significantly lower embedding capacity (10.8 KB) compared to WAV files (646 KB). Sufficient for short text payloads but not for executables or larger files.


### 7. Metadata Forensics — ExifTool Analysis

`exiftool` is used to inspect all EXIF and file header metadata of the stego-modified carrier. This is a standard first-pass forensic check when investigating suspicious media files.

> **Figure 8:** ExifTool reveals no EXIF anomalies — embedding is invisible to header-level analysis

> 🔎 **Forensic Finding:** `steghide` modifies **pixel-level LSBs** in the image data section, not the EXIF header. An analyst relying solely on `exiftool` would find nothing suspicious. Dedicated steganalysis tools (e.g., `zsteg`, `StegExpose`) or statistical analysis would be required.

<img width="360" height="79" alt="extract steghide script" src="https://github.com/user-attachments/assets/a1c70f38-7b11-44fe-89e5-a3d2ee8f6bb4" />


### 8. Filesystem Timestamps — stat Command

The `stat` command provides filesystem-level metadata including access, modification, inode change, and birth times — all relevant for forensic timeline reconstruction.

<img width="644" height="168" alt="stats analysis" src="https://github.com/user-attachments/assets/bdeeeb32-026b-47c1-82c4-de90ce2cedb6" />

> **Figure 9:** Filesystem timestamps showing ~20-minute delta between Birth and Modify times

> ⚠️ **Forensic Finding:** The **~20-minute delta** between `Birth` (23:19) and `Modify` (23:39) timestamps is consistent with the embedding operation occurring post-creation. In a real investigation, this discrepancy is an investigative lead — though it can be forged with `touch` and requires corroboration.

### 9. Extract Shell Script from WAV

`steghide extract` decrypts and recovers the embedded payload from the carrier. The `-sf` flag specifies the stego-file. After providing the correct passphrase, the original `hello.sh` is reconstructed in the working directory.

<img width="644" height="168" alt="stats analysis" src="https://github.com/user-attachments/assets/c202d4d9-5db0-419d-84b4-83c2168a970f" />

> **Figure 10:** Shell script successfully extracted from WAV carrier

### 10. Execute the Extracted Payload

The recovered shell script must be made executable with `chmod +x` before it can be run. Executing it confirms the embedded payload is fully functional.

<img width="359" height="112" alt="extracted script" src="https://github.com/user-attachments/assets/6b934150-4336-4921-8275-5b69920f3b4f" />

> **Figure 11:** Extracted shell script executed — output: `Hello World`

> 🔴 **Attack Implication:** This step demonstrates a real-world attack scenario: a threat actor could embed a **malicious shell script** inside an innocuous WAV audio file, distribute it via email or messaging, and have the recipient unknowingly extract and execute it — bypassing AV tools and content inspection systems that only analyse file headers or extensions.


### 11. Extract Text File from JPEG

Using the same extraction command against the JPEG carrier recovers the `secret.txt` payload. The `cat` command confirms the content is intact.

<img width="359" height="112" alt="extracted script" src="https://github.com/user-attachments/assets/97d36153-f366-42bb-9cbe-30fdc9dfc91f" />

> **Figure 12:** Text file extracted from JPEG carrier — content: `Confidential Notes`

<img width="359" height="112" alt="extracted script" src="https://github.com/user-attachments/assets/c2bc8326-51f9-498a-8185-338c35c3f10a" />

### 12. SHA-256 Integrity Verification

SHA-256 hashing provides definitive proof of file modification. The stego-modified `cover.jpg` is compared against an unmodified baseline `main-cover.jpg` to demonstrate that embedding always alters the binary hash — even when the visual output appears identical.

<img width="359" height="112" alt="extracted script" src="https://github.com/user-attachments/assets/c2ce1cf5-5fc4-47a9-a0c0-def48d6cdcd9" />

> **Figure 13:** Hash mismatch proves file modification — visually identical but binary-different

---

### 13. OPSEC Cleanup

After successful embedding, the original payload files are removed to reduce forensic traces — simulating the operational security (OPSEC) cleanup step an attacker would perform.

<img width="361" height="114" alt="Deleted embedded files" src="https://github.com/user-attachments/assets/a6f4a401-5435-4398-b544-fe6de2bc3561" />

> **Figure 14:** Payload originals removed — only media carrier files remain in the directory

> 📝 **Note:** The hidden payloads are **still fully preserved** inside the carriers and remain extractable at any time with the correct passphrase. The filesystem appears entirely normal to a casual observer.

---

## 🔎 Steganography vs Cryptography

| Attribute | Steganography | Cryptography |
|---|---|---|
| **Goal** | Hide the *existence* of data | Hide the *content* of data |
| **Detection Risk** | Very low — carrier looks normal | High — encrypted data is visibly scrambled |
| **Without the Key** | Investigator may not know data exists | Investigator knows data exists but cannot read it |
| **Combined Effect** | Encrypted payload embedded in carrier | — |
| **Carrier Modification** | Binary-level change; perceptually identical | N/A |

### ❌ Risks When Steganography is Used Without Encryption

| Risk | Description |
|---|---|
| **Payload Exposed on Detection** | If the carrier is found to contain data, the payload can be read directly |
| **No Confidentiality** | Any tool that extracts the data can read it without a password |
| **Single Layer of Protection** | Detection by steganalysis immediately compromises the secret |

### ✅ How Combining Both Techniques Addresses These Risks

| Improvement | Description |
|---|---|
| **Dual-Layer Protection** | Even if the carrier is identified as stego, the payload remains encrypted |
| **Passphrase Barrier** | Extraction requires both detection and the correct passphrase |
| **Plausible Deniability** | The carrier file appears innocuous to all standard analysis tools |
| **Resistance to Steganalysis** | Statistical anomalies may still be detectable, but content is inaccessible |

---

## 🔵 Detection Gaps & Countermeasures

### ❌ Detection Gaps Identified

| Gap | Detail |
|---|---|
| **`exiftool` alone insufficient** | `steghide` modifies image data, not the EXIF header — no metadata anomalies appear |
| **Timestamp delta is a weak signal** | The Birth → Modify delta can be forged with `touch`; requires corroboration |
| **Visual inspection useless** | Stego carriers are perceptually identical to clean originals |
| **AV / signature tools blind** | No malware signature exists for passphrase-encrypted embedded payloads |
| **MIME / extension checks bypass** | A malicious payload embedded in a `.wav` or `.jpg` bypasses extension-based filtering |

### ✅ Blue Team Countermeasures

| Control | Implementation |
|---|---|
| **File Integrity Monitoring (FIM)** | Maintain SHA-256 baselines for all media assets; hash mismatch = immediate flag |
| **Steganalysis Tools** | Deploy `zsteg`, `StegExpose`, or `stegdetect` for proactive scanning |
| **Statistical Analysis** | Chi-square test on pixel histograms detects LSB embedding artefacts |
| **DLP Policies** | Flag and inspect media files (`.wav`, `.jpg`) transmitted via email or external storage |
| **Deep Content Inspection** | Inspect binary structure — not just MIME type, extension, or header metadata |
| **Timestamp Correlation** | Flag unusual Birth → Modify deltas on media files during incident investigation |

---

## ✅ Verification Summary

**Step 1 — Verify embedded data in WAV carrier:**

<img width="361" height="114" alt="Deleted embedded files" src="https://github.com/user-attachments/assets/f0dfb315-5681-4526-a02d-4905b93f3ed9" />

[![steghide info output for audio.wav confirming embedded hello.sh](./assets/Audo_Steghide_info.png)](./assets/Audo_Steghide_info.png)

> **Figure 15:** Embedding confirmed in WAV — `hello.sh`, 31B, Rijndael-128 CBC, compressed

**Step 2 — Verify embedded data in JPEG carrier:**

<img width="361" height="114" alt="Deleted embedded files" src="https://github.com/user-attachments/assets/64f7f4a4-98fa-4a1c-b83a-7a2c449fe3b4" />

> **Figure 16:** Embedding confirmed in JPEG — `secret.txt`, 19B, Blowfish CBC, compressed

**Step 3 — Confirm hash mismatch between stego and clean carrier:**

<img width="361" height="114" alt="Deleted embedded files" src="https://github.com/user-attachments/assets/6a651346-4432-4871-8e45-5ea6bd681d5f" />

> **Figure 17:** SHA-256 hashes differ — file modification conclusively proven

---

## 📊 Result

| Requirement | Status |
|---|---|
| Shell script embedded in WAV using Rijndael-128 | ✅ Achieved |
| Text file embedded in JPEG using Blowfish | ✅ Achieved |
| Embedding verified with `steghide info` | ✅ Achieved |
| Carrier metadata inspected — no EXIF anomalies | ✅ Achieved |
| Both payloads successfully extracted | ✅ Achieved |
| Extracted shell script executed successfully | ✅ Achieved |
| File modification proven via SHA-256 comparison | ✅ Achieved |
| OPSEC cleanup simulated | ✅ Achieved |

The lab successfully demonstrates the complete steganography lifecycle across both audio and image carrier formats. All payloads were embedded, verified, extracted, and executed correctly. File integrity verification confirmed binary-level modification while preserving perceptual identity of both carriers.

---

## 🏁 Conclusion

This lab demonstrates the practical application of **steganography as a covert communication and payload delivery technique**, using `steghide` on Kali Linux across WAV audio and JPEG image carriers.

By completing this lab, the following key security concepts were validated:

- Steganographic embedding produces **no visible change** to carrier files yet fundamentally alters their binary structure
- Combining steganography with encryption (Rijndael-128 or Blowfish) creates a **dual-layer covert channel** resistant to both casual inspection and basic forensic analysis
- Standard metadata tools such as `exiftool` are **insufficient** to detect `steghide` embeddings — steganalysis tools and hash baselines are required
- SHA-256 hashing **conclusively proves binary modification** even when visual comparison reveals no difference
- An attacker performing OPSEC cleanup can remove all plaintext payload traces while the hidden data remains **fully recoverable** inside the carrier

> 💡 **Key Takeaway:** Never rely on visual inspection or header-only metadata analysis when investigating suspected steganographic covert channels. Deploy File Integrity Monitoring with hash baselines, steganalysis tools, and statistical content inspection as layered detection controls.

---

## 🏷️ Tags

`steganography` `steghide` `digital-forensics` `kali-linux` `steganalysis` `file-integrity-monitoring` `cryptography` `rijndael` `blowfish` `covert-channel` `exiftool` `sha256` `payload-delivery` `red-team` `blue-team` `incident-response` `opsec` `cybersecurity`

---
