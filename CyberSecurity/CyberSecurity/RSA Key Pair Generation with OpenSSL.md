# RSA Key Pair Generation with OpenSSL

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
