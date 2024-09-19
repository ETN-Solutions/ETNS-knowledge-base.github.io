---
layout: home
title: Encryption
has_children: true
---

### **Cipher Encryption**

Cipher encryption refers to the algorithms used to convert readable data (plaintext) into an unreadable format (ciphertext) to secure information. There are two main types of ciphers:

1. **Block Cipher Encryption**:
   - **Description**: Encrypts data in fixed-size blocks (e.g., 64-bit, 128-bit).
   - **How It Works**: Each block of plaintext is processed with a cryptographic key to produce a block of ciphertext. This process can use different modes of operation to enhance security (e.g., CBC, ECB, GCM).
   - **Examples**:
     - **AES (Advanced Encryption Standard)**: A widely used block cipher with 128-bit blocks.
     - **DES (Data Encryption Standard)**: An older block cipher, now considered less secure.
   - **Modes of Operation**:
     - **CBC (Cipher Block Chaining)**: Chains blocks together to improve security.
     - **ECB (Electronic Codebook)**: Simple but less secure, as identical plaintext blocks produce identical ciphertext blocks.

2. **Stream Cipher Encryption**:
   - **Description**: Encrypts data one bit or byte at a time, rather than in blocks.
   - **How It Works**: Generates a key stream and combines it with the plaintext bit-by-bit or byte-by-byte to produce ciphertext.
   - **Examples**:
     - **RC4**: A widely used stream cipher, now considered less secure.
     - **ChaCha20**: A modern, more secure stream cipher known for its speed and efficiency.

3. **Asymmetric Cipher Encryption**:
   - **Description**: Uses a pair of keys—public and private—to encrypt and decrypt data.
   - **How It Works**: The public key encrypts the data, while the private key decrypts it, allowing secure communication without sharing the private key.
   - **Examples**:
     - **RSA**: Based on the mathematical properties of prime numbers.
     - **ECC (Elliptic Curve Cryptography)**: Offers similar security to RSA but with smaller key sizes.

### **Summary**

- **Cipher Encryption**: The overarching method of securing data by transforming plaintext into ciphertext.
  - **Block Ciphers**: Work on fixed-size data blocks, with modes like CBC to enhance security.
  - **Stream Ciphers**: Encrypt data bit-by-bit or byte-by-byte, useful for real-time encryption.
  - **Asymmetric Ciphers**: Use key pairs for secure communication, eliminating the need for shared secret keys. 

Understanding these methods helps in choosing the right encryption approach based on the specific security requirements of a system.