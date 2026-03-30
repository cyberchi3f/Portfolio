# 🐦 Parrot OS Installation Guide (VMware Workstation)

## 📘 Introduction

This guide provides a complete, step-by-step walkthrough for installing **Parrot OS Security Edition 7.1 (Echo)** as a virtual machine using **VMware Workstation**.

The process is divided into two key phases:

1. **Virtual Machine Setup in VMware**
2. **Parrot OS Graphical Installation (Calamares Installer)**

Parrot OS is a Debian-based Linux distribution widely used by cybersecurity professionals, penetration testers, digital forensics analysts, and privacy-focused users.

---

## ⚙️ Prerequisites

Ensure the following requirements are met before starting:

- VMware Workstation 16.x or later
- Parrot OS ISO file: `parrot-security-7.1_amd64.iso`
- Minimum **8 GB RAM** (16 GB recommended)
- At least **80 GB free disk space**
- 64-bit CPU with virtualization enabled (Intel VT-x / AMD-V)
- VMware running with **Administrator privileges**

> ⚠️ **Warning:** Ensure virtualization is enabled in BIOS/UEFI. Without it, VMware cannot run 64-bit virtual machines.

---

# 🧩 Part 1 — Creating the Virtual Machine

## Step 1 — Select ISO Installation Source

Go to **File → New Virtual Machine**, select:

- *Installer disc image file (iso)*
- Browse and select the Parrot OS ISO

![Step 1](./images/step1.png)

> ℹ️ VMware correctly detects Parrot OS as Debian-based.

---

## Step 2 — Name the Virtual Machine

- Name: `Parrot`
- Choose storage location

![Step 2](./images/step2.png)

---

## Step 3 — Confirm ISO Selection

Verify the ISO path is correct.

![Step 3](./images/step3.png)

---

## Step 4 — Confirm VM Name

Confirm VM name and storage path.

![Step 4](./images/step4.png)

---

## Step 5 — Processor Configuration

- Processors: 1  
- Cores: 1  

![Step 5](./images/step5.png)

> 💡 Increase cores if running heavy security tools.

---

## Step 6 — Memory Allocation

- Set RAM: **4096 MB (4 GB)**

![Step 6](./images/step6.png)

> ⚠️ Do not allocate more than 50% of host RAM.

---

## Step 7 — Network Configuration

- Select: **NAT**

![Step 7](./images/step7.png)

---

## Step 8 — I/O Controller

- Select: **LSI Logic (Recommended)**

![Step 8](./images/step8.png)

---

## Step 9 — Disk Type

- Select: **SCSI (Recommended)**

![Step 9](./images/step9.png)

---

## Step 10 — Create Virtual Disk

- Choose: **Create a new virtual disk**

![Step 10](./images/step10.png)

---

## Step 11 — Disk Size

- Set: **60 GB**
- Select: *Store as a single file*

![Step 11](./images/step11.png)

> ℹ️ Disk grows dynamically unless pre-allocated.

---

## Step 12 — Disk File Name

- Default: `Parrot.vmdk`

![Step 12](./images/step12.png)

---

## Step 13 — Final Review & Create VM

- Review configuration
- Tick: *Power on after creation*
- Click **Finish**

![Step 13](./images/step13.png)

> ⚠️ VM will boot into installer immediately after this step.

---

# 💿 Part 2 — Installing Parrot OS

## Step 14 — GRUB Boot Menu

- Select: **Try / Install**

![Step 14](./images/step14.png)

---

## Step 15 — Loading Screen

Wait for Parrot OS to boot.

![Step 15](./images/step15.png)

---

## Step 16 — Launch Installer

Click **Install Parrot** on the desktop.

![Step 16](./images/step16.png)

---

## Step 17 — Open Installer

Double-click the installer icon.

![Step 17](./images/step17.png)

---

## Step 18 — Set Location

- Example:
  - Region: Africa
  - Zone: Lagos

![Step 18](./images/step18.png)

---

## Step 19 — Keyboard Layout

- Example: English (US)

![Step 19](./images/step19.png)

---

## Step 20 — Disk Partitioning

- Select: **Erase disk**
- Disk: `/dev/sda (60 GB)`
- Filesystem: **Btrfs**

![Step 20](./images/step20.png)

> ⚠️ This wipes the entire disk (safe for new VM).

---

## Step 21 — Create User Account

Fill in:

- Username
- Hostname
- Password

![Step 21](./images/step21.png)

---

## Step 22 — Confirm Installation

- Review settings
- Click **Install → Install Now**

![Step 22](./images/step22.png)

---

## Step 23 — Installation Progress

Wait for installation (5–15 minutes).

![Step 23](./images/step23.png)

---

## Step 24 — Installation Complete

- Select: **Restart now**
- Click **Done**

![Step 24](./images/step24.png)

---

## Step 25 — Login Screen

Enter credentials to log in.

![Step 25](./images/step25.png)

---

## Step 26 — Desktop Environment

You are now inside Parrot OS.

![Step 26](./images/step26.png)

---

# ✅ Post-Installation Checklist

Run the following after login:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install open-vm-tools-desktop -y
