<img width="919" height="205" alt="2026-04-04 06_24_58-KALI - VMware Workstation" src="https://github.com/user-attachments/assets/79008674-67d1-4827-a5eb-90c5e9149c29" /># RSA Key Pair Generation with OpenSSL

In this lab, you will learn about **asymmetric encryption**, also known as **public-key cryptography**. This one-way relationship is the foundation of secure communication over insecure networks.

Unlike symmetric encryption that uses only one key for both locking and unlocking messages, asymmetric encryption uses **two keys**:

- 🔓 **Public Key** → Can be shared with anyone. It is used to **lock (encrypt)** messages.
- 🔐 **Private Key** → Must stay secret. It is used to **unlock (decrypt)** messages.

This system allows people to send secret messages without needing to share a secret key in advance.

In this lab, we will use the popular **RSA** algorithm and the `openssl` tool. You will learn how to:

- Create your own pair of RSA keys
- Encrypt (lock) a message using the public key
- Decrypt (unlock) the message using the private key

## 🧰 Tools Used

- **OpenSSL**
- Linux Terminal (Kali Linux recommended)

## Step 1: Generate RSA Key Pair

Run the following command to create your RSA key pair:

```bash
openssl genrsa -out private.pem 2048

```

<img width="919" height="205" alt="2026-04-04 06_24_58-KALI - VMware Workstation" src="https://github.com/user-attachments/assets/e8460fca-b996-479e-b30a-3d72edbf4440" />

Explanation

openssl → The cryptography toolkit we are using
genrsa → Command to generate an RSA private key
-out private.pem → Saves the private key to a file named private.pem
2048 → Creates a 2048-bit key (a widely used secure key size)

After running the command, you can view the private key:
