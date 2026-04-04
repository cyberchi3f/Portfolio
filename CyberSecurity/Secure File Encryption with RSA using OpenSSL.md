 🔐 Secure File Encryption with RSA using OpenSSL

A hands-on cybersecurity lab demonstrating **asymmetric encryption using RSA** with OpenSSL.

This project walks through generating key pairs, extracting a public key, encrypting a message, and decrypting it securely using command-line tools.

---

Here's a much simpler and beginner-friendly version:

---

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

You will do everything inside your terminal, in the `~/project` folder.

---

This version uses very simple words and short sentences. Perfect for absolute beginners.

Would you like me to make it even shorter or add any examples? Just say! 😊

## 📘 Overview

Asymmetric encryption (public-key cryptography) uses **two keys**:

- 🔓 Public Key → used for encryption (shared openly)
- 🔐 Private Key → used for decryption (kept secret)

This lab demonstrates how secure communication is achieved without sharing a secret key beforehand. :contentReference[oaicite:0]{index=0}

---

## 🧰 Tools Used

- OpenSSL
- Linux Terminal (Ubuntu / Parrot OS / Kali Linux)

---

## 📂 Project Structure
