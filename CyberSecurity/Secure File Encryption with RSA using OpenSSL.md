 🔐 Secure File Encryption with RSA using OpenSSL

In this lab, you will learn about **asymmetric encryption**. 

This is an important idea in cryptography. 

Unlike normal encryption that uses only **one key** for both locking and unlocking messages, asymmetric encryption uses **two keys**: a **public key** and a **private key**.

- The **public key** can be shared with anyone. It is used to lock (encrypt) messages.  
- The **private key** must stay secret. It is used to unlock (decrypt) messages.

This system lets people send secret messages without needing to share a secret key first.

In this lab, we will use the popular **RSA** method and the `openssl` tool. You will learn how to:
- Create your own pair of keys
- Lock a message using the public key
- Unlock the message using the private key


## 📘 Overview

Asymmetric encryption (public-key cryptography) uses **two keys**:

- 🔓 Public Key → used for encryption (shared openly)
- 🔐 Private Key → used for decryption (kept secret)


---

## 🧰 Tools Used

- OpenSSL
- Linux Terminal (Kali Linux)

---

## 📂 Project Structure
