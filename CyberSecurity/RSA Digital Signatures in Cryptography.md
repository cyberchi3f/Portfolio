 🔐 Digital Signatures with RSA in Cryptography

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


ls -l
✍️ Step 1: Sign Message with Private Key

Generate a digital signature for message.txt:




🔍 Command Breakdown
openssl dgst → Performs hashing (digest operation)
-sha256 → Uses SHA-256 hashing algorithm
-sign private.pem → Signs using the private key
-out signature.bin → Outputs the signature file
message.txt → Input file to sign

Verify the signature file exists:

ls -l


✅ ## Step 2: Verify Signature with Public Key

Verify the signature using the public key:

openssl dgst -sha256 -verify public.pem -signature signature.bin message.txt
🔍 Command Breakdown
-verify public.pem → Uses public key for verification
-signature signature.bin → Signature file
message.txt → Original message

✔ Expected Output
Verified OK

✔ This confirms:

The message is authentic
The message has not been altered
📄 Step 3: Sign Another File

Sign document.txt:

openssl dgst -sha256 -sign private.pem -out document.sig document.txt

Confirm file creation:

ls
⚠️ Step 4: Test Signature Tampering

Modify the file:

echo "This is an unauthorized change." >> document.txt

Verify the modified file:

openssl dgst -sha256 -verify public.pem -signature document.sig document.txt

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
Hash Function	Security Level	Recommended
SHA-256	High	✅ Yes
SHA-3	High	✅ Yes
SHA-1	Low	❌ No
MD5	Very Low	❌ No

🛡 Why SHA-256 is Secure
Collision Resistance → Extremely unlikely two inputs share the same hash
Pre-image Resistance → Cannot reverse-engineer original data
Avalanche Effect → Tiny input change drastically alters output
Mathematical Complexity → Strong cryptographic structure


🚀 Key Takeaway

The foundation of data integrity and trust in cybersecurity, If the data changes, its cryptographic fingerprint changes — and the signature fails.


