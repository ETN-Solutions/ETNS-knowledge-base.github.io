---
layout: home
parent: Encryption
title: TS Implementation
---


```ts
async function generateECDHKeyPair(): Promise<CryptoKeyPair> {
    // Generate an ECDH key pair using the P-256 curve
    return await crypto.subtle.generateKey(
        {
            name: "ECDH",
            namedCurve: "P-256", // Can also use P-384, P-521
        },
        true, // Extractable (true) means the key can be exported
        ["deriveKey"] // The key is usable for deriving keys
    );
}

async function deriveAESKey(privateKey: CryptoKey, publicKey: CryptoKey): Promise<CryptoKey> {
    // Derive the shared secret using ECDH
    return await crypto.subtle.deriveKey(
        {
            name: "ECDH",
            public: publicKey, // The public key of the other party
        },
        privateKey, // Your own private key
        {
            name: "AES-GCM", // Specify AES-GCM for the derived key
            length: 256, // Use AES-256
        },
        true, // Extractable (true) means the key can be exported
        ["encrypt", "decrypt"] // The key is usable for encryption and decryption
    );
}


// Function to pad plaintext to be a multiple of the block size (16 bytes)
function padText(text: Uint8Array): Uint8Array {
    const blockSize = 16;
    const paddingLength = blockSize - (text.length % blockSize);
    const padding = new Uint8Array(paddingLength).fill(paddingLength);
    return new Uint8Array([...text, ...padding]);
}

// Function to remove padding from decrypted text
function unpadText(text: Uint8Array): Uint8Array {
    const paddingLength = text[text.length - 1];
    return text.slice(0, -paddingLength);
}

async function encryptAES(plaintext: string, privateKey: CryptoKey, publicKey: CryptoKey): Promise<string> {
    // Convert the plaintext to ArrayBuffer
    const plaintextBuffer = new TextEncoder().encode(plaintext);

    // Derive the AES key
    const cryptoKey = await deriveAESKey(privateKey, publicKey);

    // Generate a random 12-byte IV (AES-GCM standard)
    const iv = crypto.getRandomValues(new Uint8Array(12));

    // Encrypt the data using AES-GCM
    const encryptedBuffer = await crypto.subtle.encrypt(
        {
            name: 'AES-GCM',
            iv: iv,
        },
        cryptoKey,
        plaintextBuffer
    );

    // Convert the encrypted data and IV to base64
    const encryptedData = btoa(String.fromCharCode(...new Uint8Array(encryptedBuffer)));
    const ivBase64 = btoa(String.fromCharCode(...iv));

    // Return the IV and encrypted data concatenated with a colon
    return `${ivBase64}:${encryptedData}`;
}


async function decryptAES(encryptedData: string, privateKey: CryptoKey, publicKey: CryptoKey): Promise<string> {
    // Split the input into IV and encrypted data
    const [ivBase64, encryptedBase64] = encryptedData.split(':');

    // Decode the base64 strings
    const iv = new Uint8Array(atob(ivBase64).split('').map(c => c.charCodeAt(0)));
    const encryptedBuffer = new Uint8Array(atob(encryptedBase64).split('').map(c => c.charCodeAt(0)));

    // Derive the AES key
    const cryptoKey = await deriveAESKey(privateKey, publicKey);

    // Decrypt the data using AES-GCM
    const decryptedBuffer = await crypto.subtle.decrypt(
        {
            name: 'AES-GCM',
            iv: iv,
        },
        cryptoKey,
        encryptedBuffer
    );

    // Convert the decrypted data from ArrayBuffer to string
    return new TextDecoder().decode(decryptedBuffer);
}


// Example usage
(async () => {
    const clientKeyPair = await generateECDHKeyPair();
    const publicKeyFromServer = /* Obtain server public key as CryptoKey */;
    const plaintext = 'Hello, this is a secret message!';

    try {
        const encrypted = await encryptAES(plaintext, clientKeyPair.privateKey, publicKeyFromServer);
        console.log('Encrypted:', encrypted);

        const decrypted = await decryptAES(encrypted, clientKeyPair.privateKey, publicKeyFromServer);
        console.log('Decrypted:', decrypted);
    } catch (error) {
        console.error('Error:', error);
    }
})();

```