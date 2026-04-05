 ## 📘 Introduction

Welcome to this hands-on lab on **Digital Signatures with RSA**.

A digital signature is a cryptographic mechanism used to verify the **authenticity** and **integrity** of digital messages or documents. It ensures that:
- The message originates from a trusted sender (**authenticity**)
- The message has not been altered during transmission (**integrity**)

Digital signatures are widely used in: Secure communications (SSL/TLS), Software distribution, Blockchain systems.
In this lab, you will gain practical experience using the **OpenSSL** command-line tool to:
- Generate and use RSA keys
- Sign files using a private key
- Verify signatures using a public key
- Detect tampering through signature validation

---

##  Digital Signature Concept

Digital signatures rely on **asymmetric cryptography**, using a key pair:

- **Private Key (`private.pem`)** → Used to sign data (kept secret)
- **Public Key (`public.pem`)** → Used to verify signatures (shared)

### 🔑 Security Guarantees

- **Authenticity**: Only the private key owner can create the signature  
- **Integrity**: Any change in the data invalidates the signature  

---

## 📂 Lab Setup

The lab environment includes the following files:

- `private.pem` → Private key  
- `public.pem` → Public key  
- `message.txt` → Sample message  
- `document.txt` → Additional file  

List files in your working directory:

<img width="554" height="132" alt="image" src="https://github.com/user-attachments/assets/ea26c0da-2111-4d2f-b772-ce75584ab8f4" />

content of the text files 

<img width="542" height="121" alt="image" src="https://github.com/user-attachments/assets/e507bae3-6083-4d8d-b268-87a1020a2c59" />

ls -l

 ---

## Step 1: Sign Message with Private Key
 
Generate a digital signature for message.txt:

<img width="613" height="47" alt="image" src="https://github.com/user-attachments/assets/5d4987fc-b4eb-4a57-bf10-22eacb3e417d" />

---

 Command Breakdown
openssl dgst → Performs hashing (digest operation)

sha256 → Uses SHA-256 hashing algorithm

sign private.pem → Signs using the private key

out signature.bin → Outputs the signature file

message.txt → Input file to sign

---

Verify the signature file exists:

<img width="551" height="145" alt="image" src="https://github.com/user-attachments/assets/523432c4-8ee7-4dc2-8d79-35ff0e07a9e8" />

---

## Step 2: Verify Signature with Public Key

Verify the signature using the public key:

<img width="666" height="74" alt="image" src="https://github.com/user-attachments/assets/5906c270-33ce-4647-98d4-75bd29fd05b9" />
 ---

Command Breakdown
verify public.pem → Uses public key for verification

signature signature.bin → Signature file

message.txt → Original message

---

✔ Expected Output
Verified OK

✔ This confirms:

The message is authentic
The message has not been altered

## Step 3: Sign Another File

Sign document.txt:

<img width="659" height="61" alt="image" src="https://github.com/user-attachments/assets/17d4592b-e329-4815-8d6e-b1ccfbafcc29" />

---

Confirm file creation:

Sign File with Private Key
In this step, we will reinforce the signing process by applying it to a different file. This will help solidify your understanding of how a signed hash is created and used. We have another file in our directory named document.txt. We will now create a digital signature for this document.

The procedure is identical to what we did in Step 2. We will generate a SHA-256 hash of document.txt and then sign that hash with our private.pem key.

Let's create the signature and save it to a file named document.sig.

<img width="620" height="54" alt="image" src="https://github.com/user-attachments/assets/cd8b0de7-c4bb-41d5-95b2-4be72c3de57d" />

---

## Step 4: Test Signature Tampering

Modify the file:

<img width="710" height="55" alt="image" src="https://github.com/user-attachments/assets/bc1ceb49-368e-40c8-9fcf-530782deae0e" />

Verify the modified file:

<img width="1079" height="111" alt="image" src="https://github.com/user-attachments/assets/a2b96ecd-a9b5-4fb6-aa77-a66416166fdc" />

❌ Expected Output
Verification failure


🔬 Why Tampering Breaks the Signature
🔐 Signing Process
The file is hashed → creates a unique digital fingerprint
The hash is encrypted using the private key → signature
🔎 Verification Process
Signature is decrypted using the public key
A new hash is generated from the current file
Both hashes are compared

➡ If the file changes, the hash changes → verification fails

🧮 Understanding Hash Functions

A hash function acts as a digital fingerprint for data.

Key Properties
Deterministic → Same input produces the same output
Avalanche Effect → Small change causes large hash difference
One-Way Function → Cannot reverse hash to original data

🔑 Common Hash Functions
Hash Function	Security Level	
SHA-256	High	✅ Yes (Recommended)
SHA-3	High	✅ Yes (Recommended)
SHA-1	Low	❌ No (Not Recommended)
MD5	Very Low	❌ No (Not Recommended)

🛡 Why SHA-256 is Secure
Collision Resistance → Extremely unlikely two inputs share the same hash
Pre-image Resistance → Cannot reverse-engineer original data
Avalanche Effect → Tiny input change drastically alters output
Mathematical Complexity → Strong cryptographic structure


🚀 Key Takeaway

The foundation of data integrity and trust in cybersecurity, If the data changes, its cryptographic fingerprint changes — and the signature fails.


