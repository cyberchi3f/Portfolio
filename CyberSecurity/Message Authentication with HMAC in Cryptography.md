# HMAC — Hash-based Message Authentication Code Lab

A practical, hands-on walkthrough of HMAC (Hash-based Message Authentication Code) using the `openssl` command-line tool and Python. This lab covers the core concepts behind data integrity and authenticity, and demonstrates how HMAC protects against message tampering and forgery.

---

## What is HMAC?

A standard hash function like SHA-256 ensures **data integrity** — if a message changes, so does its hash. However, it does not guarantee **authenticity**, because an attacker can modify a message and simply recompute a new hash.

**HMAC** solves this by combining a cryptographic hash function with a **shared secret key**. Only parties who possess the secret key can generate or verify a valid HMAC for a given message.

### How HMAC Works

```
Sender:   secret_key + message → HMAC → send (message + HMAC)
Receiver: secret_key + message → HMAC → compare with received HMAC
```

| Result | Meaning |
|---|---|
| ✅ HMACs match | Message is intact and authentic |
| ❌ HMACs differ | Message was tampered with or key is wrong — discard |

---

## Lab Steps

### Step 1 — Generate the HMAC Secret Key

Generate a cryptographically secure, random 256-bit (32-byte) secret key and store it in an environment variable for use in subsequent commands.

```bash
export HMAC_KEY=$(openssl rand -hex 32)
```

**Command breakdown:**

| Flag / Argument | Description |
|---|---|
| `openssl rand` | Generates random data |
| `-hex` | Outputs the data as a hexadecimal string |
| `32` | Number of random bytes to generate (32 bytes = 256 bits) |

Verify the key was generated:

```bash
echo $HMAC_KEY
```

**Example output** (yours will be unique):

```
0db348c78473ce8460416f875cd87239d0f5f66fbe5103ba4b5c84cf2cd76914
```

> 🔑 This key is the foundation of HMAC's security. A predictable or weak key would allow an attacker to forge HMACs.

---

### Step 2 — Compute HMAC-SHA256

Create a message file and compute its HMAC using the secret key.

**Create the message file:**

```bash
echo -n "This is a secret message." > message.txt
```

> ⚠️ The `-n` flag prevents `echo` from appending a newline character, which would alter the content and change the resulting HMAC.

**Compute the HMAC:**

```bash
openssl dgst -sha256 -mac hmac -macopt hexkey:$HMAC_KEY message.txt
```

**Command breakdown:**

| Flag / Argument | Description |
|---|---|
| `openssl dgst` | Message digest (hash) utility |
| `-sha256` | Use the SHA-256 hash algorithm |
| `-mac hmac` | Compute a Message Authentication Code using HMAC |
| `-macopt hexkey:$HMAC_KEY` | Provide the key in hexadecimal format via the environment variable |
| `message.txt` | Input file to authenticate |

**Example output:**

```
HMAC-SHA2-256(message.txt)= 214fbb56f716dfe1c2f4a60615a5b901b263fd74c9678b2aef4f01779b34dea8
```

This HMAC value would be sent alongside the message to any recipient who shares the same secret key.

---

### Step 3 — Verify HMAC Integrity

This step demonstrates how HMAC detects both **wrong keys** and **tampered messages**.

#### ✅ Correct Key Verification

Re-running the same command with the same key and message will always produce the **exact same HMAC**, confirming consistency and integrity.

---

#### ❌ Failure with a Wrong Key

Simulate an incorrect key scenario:

```bash
export WRONG_KEY=$(openssl rand -hex 32)
openssl dgst -sha256 -mac hmac -macopt hexkey:$WRONG_KEY message.txt
```

**Example output:**

```
HMAC-SHA2-256(message.txt)= 781729497e89f3f9cb1d8c7ab632e22753062ac39cd64de0e5b81a45de245d78
```

The resulting HMAC is completely different. Without the correct key, it is **computationally infeasible** to produce a valid HMAC — authenticity is preserved.

---

#### ❌ Failure with a Tampered Message

Modify the message slightly:

```bash
echo -n "This is a tampered message." > message.txt
openssl dgst -sha256 -mac hmac -macopt hexkey:$HMAC_KEY message.txt
```

Again, the HMAC will be entirely different. Even a **single character change** produces a completely new digest — integrity is preserved.

---

### Step 4 — Implement HMAC in Python

Python's standard library provides the `hmac` and `hashlib` modules for programmatic HMAC operations, commonly used in web applications and APIs.

Create the script `hmac_example.py`:

```python
import hmac
import hashlib

# The secret key (must be bytes)
# In a real app, retrieve this from a secure secrets manager
key = b'\x0d\xb3\x48\xc7\x84\x73\xce\x84\x60\x41\x6f\x87\x5c\xd8\x72\x39\xd0\xf5\xf6\x6f\xbe\x51\x03\xba\x4b\x5c\x84\xcf\x2c\xd7\x69\x14'

# The message to authenticate (must be bytes)
message = b'This is a secret message.'

# Create an HMAC object and compute the digest
h = hmac.new(key, message, hashlib.sha256)
hex_digest = h.hexdigest()

print("HMAC Digest:")
print(hex_digest)
```

Run the script:

```bash
python3 hmac_example.py
```

**Expected output:**

```
HMAC Digest:
214fbb56f716dfe1c2f4a60615a5b901b263fd74c9678b2aef4f01779b34dea8
```

> 💡 Both the `key` and `message` must be **byte strings** in Python (prefixed with `b`). In production, never hardcode secrets — use a secure secrets manager or environment variables.

---

## Key Concepts Summary

| Concept | Description |
|---|---|
| **HMAC** | Combines a hash function with a secret key to ensure integrity and authenticity |
| **Integrity** | Any change to the message produces a completely different HMAC |
| **Authenticity** | Only parties with the correct secret key can generate a valid HMAC |
| **Secret Key** | A shared symmetric key — must be kept private and randomly generated |
| **SHA-256** | The underlying hash algorithm used in this lab (HMAC-SHA256) |

---

## Commands Reference

| Command | Purpose |
|---|---|
| `openssl rand -hex 32` | Generate a secure 256-bit random key in hex format |
| `openssl dgst -sha256 -mac hmac -macopt hexkey:<KEY> <file>` | Compute HMAC-SHA256 of a file |
| `python3 hmac_example.py` | Run the Python HMAC implementation |

---

## References

- [OpenSSL Documentation](https://www.openssl.org/docs/)
- [Python `hmac` Module](https://docs.python.org/3/library/hmac.html)
- [RFC 2104 — HMAC: Keyed-Hashing for Message Authentication](https://www.rfc-editor.org/rfc/rfc2104)
- [NIST FIPS 198-1 — The Keyed-Hash Message Authentication Code (HMAC)](https://csrc.nist.gov/publications/detail/fips/198/1/final)

<img width="1665" height="650" alt="image" src="https://github.com/user-attachments/assets/ce03373d-5399-4990-9fac-40d6d57b9344" />
