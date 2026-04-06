# 🔐 Securing Router Remote Access Using SSH Version 2

> **Lab Objective:** Configure secure remote administrative access to a Cisco router using Secure Shell (SSH) Version 2 — replacing insecure legacy protocols with a modern, encrypted, and resilient security standard.

---

## 📋 Table of Contents

- [Lab Environment](#lab-environment)
- [Background](#background)
- [Configuration Steps](#configuration-steps)
  - [1. Enter Global Configuration Mode](#1-enter-global-configuration-mode)
  - [2. Set the Domain Name](#2-set-the-domain-name)
  - [3. Create a Local User Account](#3-create-a-local-user-account)
  - [4. Generate RSA Cryptographic Keys](#4-generate-rsa-cryptographic-keys)
  - [5. Enable SSH Version 2](#5-enable-ssh-version-2)
  - [6. Secure SSH Sessions](#6-secure-ssh-sessions)
  - [7. Configure VTY Lines](#7-configure-vty-lines)
- [Why SSH Version 2 Over SSH Version 1.99?](#why-ssh-version-2-over-ssh-version-199)
- [Verification](#verification)
- [Result](#result)
- [Conclusion](#conclusion)

---

## 🖥️ Lab Environment

| Parameter | Value |
|---|---|
| **Device** | Cisco Router (R1) |
| **Router IP Address** | `192.168.1.1` |
| **Client Device** | Cisco Router (R2) |
| **Remote Access Protocol** | SSH Version 2 |

---

## 📖 Background

Remote administration is a critical network management function. Early implementations of SSH — such as **SSH Version 1.99** — were transitional and included legacy components from SSH Version 1, which are vulnerable to cryptographic attacks.

**SSH Version 2** was developed to address these weaknesses by introducing:
- Stronger encryption algorithms
- Improved integrity checking
- Secure key exchange mechanisms

---

## ⚙️ Configuration Steps

### 1. Enter Global Configuration Mode

The router is placed into global configuration mode to allow system-level security configurations to be applied.

```cisco
R1> enable
R1# configure terminal
R1(config)#
```

> **Figure 1:** Router R1 in Global Configuration Mode

---

### 2. Set the Domain Name

A domain name is configured to establish a unique device identity. This step is **mandatory** because SSH uses the router's Fully Qualified Domain Name (FQDN) during RSA key generation.

```cisco
R1(config)# ip domain-name example.com
```

> **Figure 2:** Configured Domain Name on Router R1

---

### 3. Create a Local User Account

A local administrative user is created with an encrypted password. This ensures authenticated and traceable access to the router when connecting via SSH.

```cisco
R1(config)# username admin privilege 15 secret <your-password>
```

> **Figure 3:** Local User Configuration on Router R1

---

### 4. Generate RSA Cryptographic Keys

RSA keys are generated to enable encrypted communication between the router and SSH clients. A **2048-bit key size** is selected to ensure security and stable performance.

```cisco
R1(config)# crypto key generate rsa modulus 2048
```

> **Figure 4:** RSA Key Generation Process

---

### 5. Enable SSH Version 2

SSH Version 2 is explicitly enabled to ensure the router does **not** fall back to legacy SSH behavior.

```cisco
R1(config)# ip ssh version 2
```

> **Figure 5:** SSH Version 2 Enabled on Router R1

---

### 6. Secure SSH Sessions

Timeout and authentication retry limits are configured to reduce exposure to **brute-force attacks** and unauthorized access attempts.

```cisco
R1(config)# ip ssh time-out 60
R1(config)# ip ssh authentication-retries 3
```

> **Figure 6:** SSH Timeout and Authentication Retry Configuration

---

### 7. Configure VTY Lines

The virtual terminal (VTY) lines are configured to accept **only SSH connections** and authenticate users using the local user database.

```cisco
R1(config)# line vty 0 4
R1(config-line)# login local
R1(config-line)# transport input ssh
R1(config-line)# exit
```

> **Figure 7:** VTY Line Configuration for SSH Access

---

## 🔎 Why SSH Version 2 Over SSH Version 1.99?

SSH Version 1.99 is a backward-compatibility mode that supports SSH Version 1 features. While it allows newer clients to connect, it exposes the system to several inherited security weaknesses.

### ❌ Vulnerabilities in SSH Version 1.99

| Vulnerability | Description |
|---|---|
| **Weak Key Exchange** | Susceptible to man-in-the-middle (MITM) attacks |
| **Poor Integrity Checking** | Allows packet manipulation in transit |
| **Session Hijacking** | Active sessions can be taken over by an attacker |
| **Deprecated Cryptography** | Uses outdated and broken cryptographic algorithms |
| **Replay Attacks** | No protection against packet replay |

### ✅ How SSH Version 2 Addresses These Issues

| Improvement | Description |
|---|---|
| **Stronger Key Exchange** | Uses Diffie-Hellman for secure key negotiation |
| **Improved Message Authentication** | Ensures message integrity via HMAC |
| **Robust Encryption** | Implements modern cipher standards |
| **Replay Attack Protection** | Built-in sequence numbering prevents replays |
| **No Legacy Fallback** | Eliminates SSH v1 compatibility entirely |

---

## ✅ Verification

**Step 1 — Check SSH Status on the Router:**

```cisco
R1# show ip ssh
```

> **Figure 8:** SSH Status Verification on Router R1

**Step 2 — Initiate SSH Connection from Client (R2):**

```cisco
R2# ssh -l admin -v 2 192.168.1.1
```

> **Figure 9:** Successful SSH Login from Client PC

---

## 📊 Result

| Security Requirement | Status |
|---|---|
| Remote connections encrypted | ✅ Achieved |
| SSH Version 2 enforced | ✅ Achieved |
| Legacy SSH vulnerabilities eliminated | ✅ Achieved |
| Access restricted to authenticated users | ✅ Achieved |
| Brute-force protection configured | ✅ Achieved |

The router successfully accepts secure remote connections using SSH Version 2. All management traffic is encrypted, legacy SSH vulnerabilities are eliminated, and access is restricted to authenticated users only.

---

## 🏁 Conclusion

This lab demonstrates the implementation of secure remote management using **SSH Version 2**, highlighting the critical importance of disabling legacy compatibility modes such as SSH Version 1.99.

By enforcing modern cryptographic standards, generating 2048-bit RSA keys, and restricting VTY access to SSH-only connections, the router is effectively protected against:

- Man-in-the-middle (MITM) attacks
- Brute-force login attempts
- Session hijacking
- Replay attacks
- Cryptographic downgrade attacks

> 💡 **Key Takeaway:** Never rely on backward-compatibility modes in production environments. Always enforce the most current and secure version of any protocol to minimise your attack surface.

---

## 🏷️ Tags

`cisco` `networking` `ssh` `cybersecurity` `router-security` `cryptography` `rsa` `remote-access` `vty` `ccna` `network-hardening`

---

*Lab documented as part of a cybersecurity portfolio. All configurations were implemented in a controlled lab environment.*
