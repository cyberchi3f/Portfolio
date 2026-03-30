# 🐦 Parrot OS Installation Guide (VMware Workstation)

## 📘 Introduction

This guide provides a complete, step-by-step walkthrough for installing **Parrot OS Security Edition 7.1 (Echo)** as a virtual machine using **VMware Workstation**.

The process is divided into two key phases:

1. **Virtual Machine Setup in VMware**
2. **Parrot OS Graphical Installation (Calamares Installer)**

Parrot OS is a Debian-based Linux distribution widely used by cybersecurity professionals, penetration testers, digital forensics analysts, and privacy-focused users.


## ⚙️ Prerequisites

Ensure the following requirements are met before starting:

- VMware Workstation 16.x or later
- Parrot OS ISO file: `parrot-security-7.1_amd64.iso`
- Minimum **8 GB RAM** (16 GB recommended)
- At least **80 GB free disk space**
- 64-bit CPU with virtualization enabled (Intel VT-x / AMD-V)
- VMware running with **Administrator privileges**

> ⚠️ **Warning:** Ensure virtualization is enabled in BIOS/UEFI. Without it, VMware cannot run 64-bit virtual machines.



# 🧩 Part 1 — Creating the Virtual Machine

## Step 1 — Select ISO Installation Source

Go to **File → New Virtual Machine**, select:

- *Installer disc image file (iso)*
- Browse and select the Parrot OS ISO

<img width="430" height="430" alt="2026-03-27 13_07_07-Sudo - VMware Workstation" src="https://github.com/user-attachments/assets/17ec715c-cfda-40e7-808b-7f7850f6f50e" />


> ℹ️ VMware correctly detects Parrot OS as Debian-based.

## Step 2 — Name the Virtual Machine

- Name: `Parrot`
- Choose storage location

<img width="426" height="427" alt="2026-03-27 13_07_48-New Virtual Machine Wizard" src="https://github.com/user-attachments/assets/223bfb8f-e70a-4875-9efb-4864d0e4315b" />



## Step 3 — Confirm ISO Selection

Verify the ISO path is correct.

<img width="430" height="430" alt="2026-03-27 13_12_39-Sudo - VMware Workstation" src="https://github.com/user-attachments/assets/3e5b44b4-3fd2-4b6c-aac3-d83ec4ec4d9b" />


## Step 4 — Confirm VM Name

Confirm VM name and storage path.

<img width="428" height="428" alt="2026-03-27 13_14_51-New Virtual Machine Wizard" src="https://github.com/user-attachments/assets/1794bd5f-7a16-4e1a-b5ed-a475f55526cf" />


## Step 5 — Processor Configuration

- Processors: 1  
- Cores: 1  

<img width="425" height="428" alt="2026-03-27 13_16_15-Sudo - VMware Workstation" src="https://github.com/user-attachments/assets/d9b87454-85fd-4407-877a-2537defdb62d" />


> 💡 Increase cores if running heavy security tools.

## Step 6 — Memory Allocation

- Set RAM: **4096 MB (4 GB)**

<img width="426" height="428" alt="2026-03-27 13_17_21-New Virtual Machine Wizard" src="https://github.com/user-attachments/assets/afce9886-43fe-471b-b4d5-dd221d86d53c" />

> ⚠️ Do not allocate more than 50% of host RAM.



## Step 7 — Network Configuration

- Select: **NAT**

<img width="435" height="430" alt="2026-03-27 13_18_15-Sudo - VMware Workstation" src="https://github.com/user-attachments/assets/fda5b22b-feb1-4219-a5f4-ac003de18d21" />


## Step 8 — I/O Controller

- Select: **LSI Logic (Recommended)**

<img width="424" height="434" alt="2026-03-27 13_18_55-Sudo - VMware Workstation" src="https://github.com/user-attachments/assets/0a33ac26-15a2-4263-8a67-b4fafae6daa7" />

## Step 9 — Disk Type

- Select: **SCSI (Recommended)**

<img width="424" height="434" alt="2026-03-27 13_18_55-Sudo - VMware Workstation" src="https://github.com/user-attachments/assets/c6ad9377-3f40-412a-bc37-484da2ba355f" />


## Step 10 — Create Virtual Disk

- Choose: **Create a new virtual disk**

<img width="424" height="428" alt="2026-03-27 13_19_40-New Virtual Machine Wizard" src="https://github.com/user-attachments/assets/cafc556f-0381-444e-ac45-01b5c595e7e5" />


## Step 11 — Disk Size

- Set: **60 GB**
- Select: *Store as a single file*

<img width="421" height="427" alt="2026-03-27 13_20_31-Sudo - VMware Workstation" src="https://github.com/user-attachments/assets/64051f20-36fa-4af3-b6b8-ebcd225fcd56" />


> ℹ️ Disk grows dynamically unless pre-allocated.

## Step 12 — Disk File Name

- Default: `Parrot.vmdk`

<img width="424" height="424" alt="2026-03-27 13_23_33-New Virtual Machine Wizard" src="https://github.com/user-attachments/assets/6aff032b-5fda-447f-849d-f838cb5cd8f2" />


## Step 13 — Final Review & Create VM

- Review configuration
- Tick: *Power on after creation*
- Click **Finish**

<img width="422" height="423" alt="2026-03-27 13_25_13-New Virtual Machine Wizard" src="https://github.com/user-attachments/assets/9abb5e1f-6342-4c6c-bff7-9911d60761b4" />


> ⚠️ VM will boot into installer immediately after this step.

# 💿 Part 2 — Installing Parrot OS

## Step 14 — GRUB Boot Menu

- Select: **Try / Install**

<img width="732" height="421" alt="2026-03-27 13_26_22-Parrot - VMware Workstation" src="https://github.com/user-attachments/assets/8a01aff9-5ab3-4a63-b63b-94b0ce026b03" />


## Step 15 — Loading Screen

Wait for Parrot OS to boot.

<img width="1245" height="732" alt="2026-03-27 13_27_05-Parrot - VMware Workstation" src="https://github.com/user-attachments/assets/a05e9540-f394-4f58-9247-eaa574acbd1a" />


## Step 16 — Launch Installer

Click **Install Parrot** on the desktop.

## Step 17 — Open Installer
<img width="1366" height="768" alt="2026-03-27 13_31_48-What Is HSTS and How Do You Set It Up_ and 21 more pages - Personal - Microsoft​" src="https://github.com/user-attachments/assets/77a30e8b-7b22-4130-a45d-1d49e49d3182" />

Double-click the installer icon.


## Step 18 — Set Location

- Example:
  - Region: Africa
  - Zone: Lagos

<img width="1038" height="610" alt="2026-03-27 13_36_14-Parrot - VMware Workstation" src="https://github.com/user-attachments/assets/d0eef932-0a45-4964-893c-7c8957efad96" />


## Step 19 — Keyboard Layout

- Example: English (US)


<img width="1038" height="610" alt="2026-03-27 13_36_58-Parrot - VMware Workstation" src="https://github.com/user-attachments/assets/4cdcbd26-f085-4115-8a4b-ea9b0cb44353" />


## Step 20 — Disk Partitioning

- Select: **Erase disk**
- Disk: `/dev/sda (60 GB)`
- Filesystem: **Btrfs**

<img width="1038" height="610" alt="2026-03-27 13_37_38-Parrot - VMware Workstation" src="https://github.com/user-attachments/assets/3dab1e03-a8f2-4ad1-afd1-f8f3741da055" />


> ⚠️ This wipes the entire disk (safe for new VM).


## Step 21 — Create User Account

Fill in:

- Username
- Hostname
- Password

<img width="1038" height="610" alt="2026-03-27 13_39_05-Parrot - VMware Workstation" src="https://github.com/user-attachments/assets/7d6766be-a7e6-4a30-a8b3-9769bd4b5e45" />


## Step 22 — Confirm Installation

- Review settings
- Click **Install → Install Now**

<img width="1022" height="694" alt="2026-03-27 13_40_17-Parrot - VMware Workstation" src="https://github.com/user-attachments/assets/45559875-6084-4b66-a625-06f5a242d17b" />

## Step 23 — Installation Progress

Wait for installation (5–15 minutes).

<img width="1056" height="691" alt="2026-03-30 08_35_10-Clipboard" src="https://github.com/user-attachments/assets/1adc44bc-1e1d-4ede-ae31-3de34f28681d" />


## Step 24 — Installation Complete

- Select: **Restart now**
- Click **Done**

<img width="1031" height="703" alt="2026-03-30 09_29_04-Parrot - VMware Workstation" src="https://github.com/user-attachments/assets/9119ea7e-f46c-4247-8b2a-bbf7e78316b7" />

## Step 25 — Login Screen

Enter credentials to log in.

<img width="1366" height="768" alt="2026-03-30 09_31_14-Parrot - VMware Workstation" src="https://github.com/user-attachments/assets/e7601f60-44eb-4315-88b6-3db3e65e2c45" />

## Step 26 — Desktop Environment

You are now inside Parrot OS.

<img width="1360" height="771" alt="2026-03-30 10_52_34-" src="https://github.com/user-attachments/assets/e25f88fe-9b4d-4345-a846-2f487b13ac92" />
