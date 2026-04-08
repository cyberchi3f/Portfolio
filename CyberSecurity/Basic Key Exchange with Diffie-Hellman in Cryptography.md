# Diffie-Hellman Key Exchange Lab

A hands-on simulation of the Diffie-Hellman (DH) key exchange protocol using the `openssl` command-line tool. This lab walks through generating shared parameters, creating key pairs for two parties, and independently deriving an identical shared secret — the core proof of concept behind DH.

---

## Prerequisites

- OpenSSL installed and available in your terminal
- All commands are executed from the default `~/project` directory

---

## Lab Steps

### Step 1 — Generate Diffie-Hellman Parameters

Generate the shared public DH parameters (`p` and `g`) that both parties will use. OpenSSL handles the complex prime-finding mathematics internally.

```bash
openssl dhparam -out dhparam.pem 2048
```

**Command breakdown:**

| Flag / Argument | Description |
|---|---|
| `openssl dhparam` | Invokes the DH parameter management tool |
| `-out dhparam.pem` | Saves output to `dhparam.pem` |
| `2048` | Bit length of the prime modulus `p` |

> ⏳ This command may take a minute to complete. You will see the following while it works:
> ```
> Generating DH parameters, 2048 bit long safe prime
> ```

**Output:** A file named `dhparam.pem` containing the public parameters both parties will use.

---

### Step 2 — Generate Party A's Keys

Simulate **Party A** generating its private key (secret `a`) and deriving a corresponding public key.

**Generate Party A's private key:**

```bash
openssl genpkey -paramfile dhparam.pem -out a_private_key.pem
```

| Flag / Argument | Description |
|---|---|
| `genpkey` | General-purpose key generation utility |
| `-paramfile dhparam.pem` | Uses the shared DH parameters from Step 1 |
| `-out a_private_key.pem` | Saves the private key to `a_private_key.pem` |

**Extract Party A's public key:**

```bash
openssl pkey -in a_private_key.pem -pubout -out a_public_key.pem
```

| Flag / Argument | Description |
|---|---|
| `pkey` | Key management command |
| `-in a_private_key.pem` | Specifies the input private key |
| `-pubout` | Outputs the public part of the key |
| `-out a_public_key.pem` | Saves the public key to `a_public_key.pem` |

**Output files:**

| File | Visibility |
|---|---|
| `a_private_key.pem` | 🔒 Keep secret — never share |
| `a_public_key.pem` | 🌐 Share with Party B over the insecure channel |

---

### Step 3 — Generate Party B's Keys

Perform the same key generation process for **Party B**, independently, using the same shared `dhparam.pem`.

**Generate Party B's private key:**

```bash
openssl genpkey -paramfile dhparam.pem -out b_private_key.pem
```

**Extract Party B's public key:**

```bash
openssl pkey -in b_private_key.pem -pubout -out b_public_key.pem
```

**Output files:**

| File | Visibility |
|---|---|
| `b_private_key.pem` | 🔒 Keep secret — never share |
| `b_public_key.pem` | 🌐 Share with Party A over the insecure channel |

> At this point, Party A holds `b_public_key.pem`, and Party B holds `a_public_key.pem`. Both private keys remain secret. The public exchange has been simulated.

---

### Step 4 — Compute the Shared Secret

Both parties now independently derive a shared secret using their own private key and the other party's public key. A successful exchange means both arrive at **the exact same value**.

**Party A computes the shared secret:**

```bash
openssl pkeyutl -derive -inkey a_private_key.pem -peerkey b_public_key.pem -out a_shared_secret.bin
```

**Party B computes the shared secret:**

```bash
openssl pkeyutl -derive -inkey b_private_key.pem -peerkey a_public_key.pem -out b_shared_secret.bin
```

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

```bash
cmp a_shared_secret.bin b_shared_secret.bin
```

> ✅ **No output = success.** Silence means the files are byte-for-byte identical.

For a visual confirmation, compare the SHA-256 hashes of both files:

```bash
sha256sum *.bin
```

**Expected output format** (your actual hashes will differ, but they must match each other):

```
e3705a4ab5ae5d86f59dfe968f0177b49d5144e2d731dbd8d41b2eda318412ec  a_shared_secret.bin
e3705a4ab5ae5d86f59dfe968f0177b49d5144e2d731dbd8d41b2eda318412ec  b_shared_secret.bin
```

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




<img width="1020" height="538" alt="image" src="https://github.com/user-attachments/assets/04de1d57-9a7e-47c7-9640-71dcaf032875" />


<img width="801" height="71" alt="image" src="https://github.com/user-attachments/assets/1a0f2a87-e9c9-4fca-94c7-60a30c767519" />


<img width="829" height="78" alt="image" src="https://github.com/user-attachments/assets/943a1e62-905c-46ca-bace-ec20f9429c11" />

<img width="819" height="78" alt="image" src="https://github.com/user-attachments/assets/7dc3eec8-5f9f-4498-a28c-853cc30f288f" />


<img width="835" height="75" alt="image" src="https://github.com/user-attachments/assets/486f5606-ecc8-4e16-8cf7-31af7a3419c1" />

<img width="1238" height="73" alt="image" src="https://github.com/user-attachments/assets/4d3e3298-5fde-45e5-b0a9-1727f33e7065" />


<img width="1254" height="72" alt="image" src="https://github.com/user-attachments/assets/fdb7328a-1353-497c-bb5d-dffb33837666" />

<img width="591" height="81" alt="image" src="https://github.com/user-attachments/assets/7610d0eb-8055-42b4-8d45-d6699703ef9c" />

<img width="1063" height="126" alt="image" src="https://github.com/user-attachments/assets/eb8a5d10-7a9c-4a55-931d-493b2714493c" />
