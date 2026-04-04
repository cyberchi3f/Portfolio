 🔐 Secure File Encryption with RSA using OpenSSL

In this lab, you will learn about **asymmetric encryption**. also known as public-key cryptography. This one-way relationship is the foundation of secure communication over insecure networks. 

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

## Step 1: Generate RSA Key Pair

Run this command to create your RSA key pair:


openssl genrsa -out private.pem 2048

Explanation

openssl → The tool we are using
genrsa → Command to generate an RSA private key
-out private.pem → Saves the key to a file named private.pem
2048 → Makes a 2048-bit key (a common secure size)
You should see private.pem in the list.


##Step 2: Extract the Public Key
The private.pem file contains both private and public parts. Now extract the public key so others can use it:

Explanation

openssl rsa → Tool for working with RSA keys
-in private.pem → Uses your private key file
-pubout → Outputs only the public key
-out public.pem → Saves it as public.pem

You should now see private.pem and public.pem.

##Step 3: Encrypt a Message with the Public Key

First, create a simple secret message:

Now encrypt it using the public key:

Explanation

openssl pkeyutl → Tool for public key operations (encrypt/decrypt)
-encrypt → We want to encrypt
-pubin → Tells OpenSSL we are using a public key
-inkey public.pem → Uses your public key
-in message.txt → The message to encrypt
-out encrypted.bin → Saves the encrypted result

The file encrypted.bin will look like unreadable binary data if you try to open it.

##Step 4: Decrypt the Message with the Private Key
Decrypt the encrypted file using your private key:
Bash

Explanation

-decrypt → We want to decrypt
-inkey private.pem → Uses your private key
-in encrypted.bin → The encrypted file
-out decrypted.txt → Saves the result as text

Check the decrypted message:

You should see:



You have successfully completed a full asymmetric encryption cycle:

Generated an RSA key pair
Encrypted a message with the public key (anyone can do this)
Decrypted it with the private key (only you can do this)

This is how secure communication works on the internet — for example, in HTTPS websites, secure email, and digital signatures.
