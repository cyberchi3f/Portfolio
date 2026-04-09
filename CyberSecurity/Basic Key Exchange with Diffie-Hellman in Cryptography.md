# Diffie-Hellman Key Exchange Lab

A hands-on simulation of the Diffie-Hellman (DH) key exchange protocol using the `openssl` command-line tool. This lab walks through generating shared parameters, creating key pairs for two parties, and independently deriving an identical shared secret — the core proof of concept behind DH.

---

## Prerequisites

- OpenSSL installed and available in your terminal

---

## Lab Steps

### Step 1 — Generate Diffie-Hellman Parameters

Generate the shared public DH parameters (`p` and `g`) that both parties will use. OpenSSL handles the complex prime-finding mathematics internally.

<img width="1020" height="538" alt="image" src="https://github.com/user-attachments/assets/04de1d57-9a7e-47c7-9640-71dcaf032875" />

**Command breakdown:**

| Flag / Argument | Description |
|---|---|
| `openssl dhparam` | Invokes the DH parameter management tool |
| `-out dhparam.pem` | Saves output to `dhparam.pem` |
| `2048` | Bit length of the prime modulus `p` |

>  This command may take a minute to complete. You will see the following while it works:
> ```
> Generating DH parameters, 2048 bit long safe prime
> ```

**Output:** A file named `dhparam.pem` containing the public parameters both parties will use.

<img width="413" height="86" alt="image" src="https://github.com/user-attachments/assets/3c944e10-893c-40fc-9915-b2d0aacd3abd" />

---

### Step 2 — Generate Party A's Keys

Simulate **Party A** generating its private key (secret `a`) and deriving a corresponding public key.

**Generate Party A's private key:**

<img width="796" height="68" alt="image" src="https://github.com/user-attachments/assets/ee880e9f-9eb4-4add-98af-9b78298bad00" />


| Flag / Argument | Description |
|---|---|
| `genpkey` | General-purpose key generation utility |
| `-paramfile dhparam.pem` | Uses the shared DH parameters from Step 1 |
| `-out a_private_key.pem` | Saves the private key to `a_private_key.pem` |

**Extract Party A's public key:**

<img width="837" height="84" alt="image" src="https://github.com/user-attachments/assets/653cde4b-cf64-467b-bcb5-5f51b2026a20" />


| Flag / Argument | Description |
|---|---|
| `pkey` | Key management command |
| `-in a_private_key.pem` | Specifies the input private key |
| `-pubout` | Outputs the public part of the key |
| `-out a_public_key.pem` | Saves the public key to `a_public_key.pem` |

**Output files:**

<img width="633" height="98" alt="image" src="https://github.com/user-attachments/assets/e94fd19a-b868-4999-9743-10aa9f03a29e" />

| File | Visibility |
|---|---|
| `a_private_key.pem` | 🔒 Keep secret — never share |
| `a_public_key.pem` | 🌐 Share with Party B over the insecure channel |

---

### Step 3 — Generate Party B's Keys

Perform the same key generation process for **Party B**, independently, using the same shared `dhparam.pem`.

**Generate Party B's private key:**

<img width="805" height="71" alt="image" src="https://github.com/user-attachments/assets/6de80259-aa72-4711-9ba4-d9f534a3c1cc" />

**Extract Party B's public key:**

<img width="835" height="80" alt="image" src="https://github.com/user-attachments/assets/3a0ea7a3-f89e-4e39-9855-11b72b5d616c" />

**Output files:**

<img width="1064" height="99" alt="image" src="https://github.com/user-attachments/assets/7d72555d-1750-479b-b791-1bf173de4107" />

| File | Visibility |
|---|---|
| `b_private_key.pem` | 🔒 Keep secret — never share |
| `b_public_key.pem` | 🌐 Share with Party A over the insecure channel |

> At this point, Party A holds `b_public_key.pem`, and Party B holds `a_public_key.pem`. Both private keys remain secret. The public exchange has been simulated.

---

### Step 4 — Compute the Shared Secret

Both parties now independently derive a shared secret using their own private key and the other party's public key. A successful exchange means both arrive at **the exact same value**.

**Party A computes the shared secret:**

<img width="1252" height="72" alt="image" src="https://github.com/user-attachments/assets/490d51b6-ca71-4fb2-8726-ecc645f1fc71" />

**Party B computes the shared secret:**

<img width="1267" height="75" alt="image" src="https://github.com/user-attachments/assets/a5344617-92f2-4754-a68e-0e2dc7c9f07b" />

**Output files:**

<img width="1545" height="92" alt="image" src="https://github.com/user-attachments/assets/f18854b0-0828-4bd3-bfe1-be619452f974" />

**Command breakdown:**

| Flag / Argument | Description |
|---|---|
| `pkeyutl` | Utility for public key operations |
| `-derive` | Instructs the tool to derive a shared secret |
| `-inkey` | The party's own private key |
| `-peerkey` | The other party's public key |
| `-out` | Saves the binary secret to a file |

---

### Step 5 — Verify the Shared Secret

Use `cmp` to confirm both secret files are identical:

<img width="605" height="68" alt="image" src="https://github.com/user-attachments/assets/3953d720-3d2b-4ee8-84ff-d83a106b03a6" />

> ✅ **No output = success.** Silence means the files are byte-for-byte identical.

For a visual confirmation, compare the SHA-256 hashes of both files:
<img width="406" height="64" alt="image" src="https://github.com/user-attachments/assets/93fb9c5a-420c-4fec-b5ec-80ea7345e867" />


**Expected output format** (your actual hashes will differ, but they must match each other):

<img width="1057" height="152" alt="image" src="https://github.com/user-attachments/assets/c8853b6d-69dc-4c2b-8519-eacbe56f7d83" />

Matching hashes confirm that both parties independently derived the **same shared secret** — the core proof of a successful Diffie-Hellman exchange.

---

## File Summary

| File | Description |
|---|---|
| `dhparam.pem` | Shared public DH parameters (prime `p` and generator `g`) |
| `a_private_key.pem` | Party A's secret private key |
| `a_public_key.pem` | Party A's public key (shared with Party B) |
| `b_private_key.pem` | Party B's secret private key |
| `b_public_key.pem` | Party B's public key (shared with Party A) |
| `a_shared_secret.bin` | Shared secret derived from Party A's perspective |
| `b_shared_secret.bin` | Shared secret derived from Party B's perspective |

---

## Key Takeaways

- **`openssl dhparam`** — Generates shared public DH parameters
- **`openssl genpkey`** — Creates a private key from DH parameters
- **`openssl pkey -pubout`** — Derives a public key from a private key
- **`openssl pkeyutl -derive`** — Computes the shared secret using one's own private key and a peer's public key

This protocol is a foundational building block of secure communication systems, including **TLS/SSL**, which protects data across the internet. Two parties can establish a secure channel even when communicating over an entirely insecure network — without ever transmitting the secret itself.

---

## References

- [OpenSSL Documentation](https://www.openssl.org/docs/)
- [RFC 3526 — More Modular Exponential (MODP) Diffie-Hellman groups](https://www.rfc-editor.org/rfc/rfc3526)
- [NIST SP 800-56A — Recommendation for Pair-Wise Key-Establishment Schemes](https://csrc.nist.gov/publications/detail/sp/800-56a/rev-3/final)



