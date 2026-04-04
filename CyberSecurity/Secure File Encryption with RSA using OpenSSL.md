 # 🔐 Secure File Encryption with RSA using OpenSSL

In this lab, you will learn about **asymmetric encryption**. also known as public-key cryptography. This one-way relationship is the foundation of secure communication over insecure networks. 

Unlike normal encryption that uses only **one key** for both locking and unlocking messages, asymmetric encryption uses **two keys**: a **public key** and a **private key**.

- 🔓 Public Key → can be shared with anyone. It is used to lock (encrypt) messages.
- 🔐 Private Key → must stay secret. It is used to unlock (decrypt) messages.

This system lets people send secret messages without needing to share a secret key first.

In this lab, we will use the popular **RSA** method and the `openssl` tool. You will learn how to:
- Create your own pair of keys
- Lock a message using the public key
- Unlock the message using the private key

## 🧰 Tools Used

- OpenSSL
- Linux Terminal (Kali Linux)

---

## Step 1: Generate RSA Key Pair

Run this command to create your RSA key pair:

<img width="369" height="44" alt="image" src="https://github.com/user-attachments/assets/97b6e242-0ab2-491b-94ef-2354dfd6a174" />

---

Explanation

openssl → The tool we are using

genrsa → Command to generate an RSA private key

-out private.pem → Saves the key to a file named private.pem

2048 → Makes a 2048-bit key (a common secure size)

You read the private.pem 

<img width="496" height="457" alt="image" src="https://github.com/user-attachments/assets/b6127709-32ea-46ab-8424-fad54ca66ce5" />

---

## Step 2: Extract the Public Key

The private.pem file contains both private and public parts. Now extract the public key so others can use it:

<img width="560" height="258" alt="2026-04-04 06_22_31-KALI - VMware Workstation" src="https://github.com/user-attachments/assets/f3d002b2-2088-4794-aaf5-2ee2e19c87bc" />

Explanation

openssl rsa → Tool for working with RSA keys

-in private.pem → Uses your private key file

-pubout → Outputs only the public key

-out public.pem → Saves it as public.pem

You should now see private.pem and public.pem.

<img width="402" height="61" alt="image" src="https://github.com/user-attachments/assets/d17e02a1-3073-4923-a9f6-bd570ce4f0f5" />


---

## Step 3: Encrypt a Message with the Public Key


First, create a simple secret message:

<img width="420" height="59" alt="image" src="https://github.com/user-attachments/assets/877809ae-a01a-40c1-81a5-ee864a41dd43" />

Now encrypt it using the public key:

<img width="803" height="50" alt="image" src="https://github.com/user-attachments/assets/628a7d56-1a19-4b6c-a217-f3b1452c37df" />

Explanation

openssl pkeyutl → Tool for public key operations (encrypt/decrypt)

-encrypt → We want to encrypt

-pubin → Tells OpenSSL we are using a public key

-inkey public.pem → Uses your public key

-in message.txt → The message to encrypt

-out encrypted.bin → Saves the encrypted result

The file encrypted_message.txt will look like unreadable binary data if you try to open it.

<img width="958" height="147" alt="image" src="https://github.com/user-attachments/assets/51460c7e-8083-4e59-9dbc-a36e5118d73e" />

---

## Step 4: Decrypt the Message with the Private Key
Decrypt the encrypted file using your private key:

<img width="822" height="54" alt="image" src="https://github.com/user-attachments/assets/e487e54e-a2ce-498c-9f9c-1fdbabe14e9d" />


Explanation

-decrypt → We want to decrypt

-inkey private.pem → Uses your private key

-in encrypted.bin → The encrypted file

-out decrypted.txt → Saves the result as text

Check the decrypted message:

You should see:

<img width="438" height="67" alt="image" src="https://github.com/user-attachments/assets/0ee44ddf-a02b-4ae8-9009-eb624a6b45c4" />


You have successfully completed a full asymmetric encryption cycle:

Generated an RSA key pair
Encrypted a message with the public key (anyone can do this)
Decrypted it with the private key (only you can do this)

This is how secure communication works on the internet — for example, in HTTPS websites, secure email, and digital signatures.
