---
layout: home
parent: Encryption
title: Spring Boot Implementation
---

### Spring Boot Implementation

```java
import org.bouncycastle.jce.provider.BouncyCastleProvider;
import org.bouncycastle.jce.spec.ECGenParameterSpec;
import org.bouncycastle.util.encoders.Base64;

import javax.crypto.Cipher;
import javax.crypto.SecretKey;
import javax.crypto.KeyAgreement;
import javax.crypto.spec.GCMParameterSpec;
import javax.crypto.spec.SecretKeySpec;
import java.security.*;
import java.util.Arrays;

public class CryptoUtils {
    
    // Register Bouncy Castle as a security provider
    static {
        Security.addProvider(new BouncyCastleProvider());
    }

    /**
     * Generate an ECDH key pair using the P-256 curve.
     * @return KeyPair containing private and public keys.
     * @throws Exception if key generation fails.
     */
    public static KeyPair generateECDHKeyPair() throws Exception {
        KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance("EC", "BC");
        keyPairGenerator.initialize(new ECGenParameterSpec("P-256"), new SecureRandom());
        return keyPairGenerator.generateKeyPair();
    }

    /**
     * Derive an AES key from the shared secret generated using ECDH.
     * @param privateKey Your private ECDH key.
     * @param publicKey The public key of the other party.
     * @return SecretKey derived from the shared secret.
     * @throws Exception if key derivation fails.
     */
    public static SecretKey deriveAESKey(PrivateKey privateKey, PublicKey publicKey) throws Exception {
        KeyAgreement keyAgreement = KeyAgreement.getInstance("ECDH", "BC");
        keyAgreement.init(privateKey);
        keyAgreement.doPhase(publicKey, true);

        byte[] sharedSecret = keyAgreement.generateSecret();
        return new SecretKeySpec(Arrays.copyOf(sharedSecret, 32), "AES"); // Use 256 bits for AES-256
    }

    /**
     * Encrypt plaintext using AES-GCM.
     * @param plaintext The plaintext to encrypt.
     * @param secretKey The AES secret key.
     * @return Base64-encoded ciphertext combined with the IV.
     * @throws Exception if encryption fails.
     */
    public static String encryptAES(String plaintext, SecretKey secretKey) throws Exception {
        Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");
        byte[] iv = new byte[12]; // 12-byte IV for AES-GCM
        SecureRandom random = new SecureRandom();
        random.nextBytes(iv); // Generate random IV
        
        GCMParameterSpec gcmParameterSpec = new GCMParameterSpec(128, iv); // 128 bits for authentication tag
        cipher.init(Cipher.ENCRYPT_MODE, secretKey, gcmParameterSpec);
        
        byte[] ciphertext = cipher.doFinal(plaintext.getBytes());
        
        // Combine IV and ciphertext
        byte[] combined = new byte[iv.length + ciphertext.length];
        System.arraycopy(iv, 0, combined, 0, iv.length);
        System.arraycopy(ciphertext, 0, combined, iv.length, ciphertext.length);
        
        return Base64.encode(combined); // Return Base64-encoded combined data
    }

    /**
     * Decrypt ciphertext using AES-GCM.
     * @param encryptedData Base64-encoded data containing IV and ciphertext.
     * @param secretKey The AES secret key.
     * @return Decrypted plaintext.
     * @throws Exception if decryption fails.
     */
    public static String decryptAES(String encryptedData, SecretKey secretKey) throws Exception {
        byte[] combined = Base64.decode(encryptedData); // Decode Base64 data
        byte[] iv = Arrays.copyOfRange(combined, 0, 12); // Extract IV (first 12 bytes)
        byte[] ciphertext = Arrays.copyOfRange(combined, 12, combined.length); // Extract ciphertext (remaining bytes)
        
        Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");
        GCMParameterSpec gcmParameterSpec = new GCMParameterSpec(128, iv);
        cipher.init(Cipher.DECRYPT_MODE, secretKey, gcmParameterSpec);
        
        byte[] decrypted = cipher.doFinal(ciphertext); // Decrypt ciphertext
        return new String(decrypted); // Return the decrypted plaintext
    }
}
```

### Updated Controller

```java
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/crypto")
public class CryptoController {

    /**
     * Generate a new ECDH key pair.
     * @return Base64-encoded private and public keys.
     * @throws Exception if key generation fails.
     */
    @GetMapping("/generateKeys")
    public String generateKeys() throws Exception {
        KeyPair keyPair = CryptoUtils.generateECDHKeyPair();
        return "Private Key: " + Base64.encode(keyPair.getPrivate().getEncoded()) +
               ", Public Key: " + Base64.encode(keyPair.getPublic().getEncoded());
    }

    /**
     * Encrypt plaintext using AES-GCM.
     * @param plaintext The plaintext to encrypt.
     * @param publicKeyEncoded The public key of the other party (Base64 encoded).
     * @param privateKeyEncoded Your private key (Base64 encoded).
     * @return Encrypted data (Base64 encoded).
     * @throws Exception if encryption fails.
     */
    @PostMapping("/encrypt")
    public String encrypt(@RequestBody String plaintext, @RequestParam String publicKeyEncoded, 
                          @RequestParam String privateKeyEncoded) throws Exception {
        // Decode Base64 keys
        PrivateKey privateKey = KeyFactory.getInstance("EC", "BC")
                .generatePrivate(new PKCS8EncodedKeySpec(Base64.decode(privateKeyEncoded)));
        PublicKey publicKey = KeyFactory.getInstance("EC", "BC")
                .generatePublic(new X509EncodedKeySpec(Base64.decode(publicKeyEncoded)));
        
        // Derive AES key
        SecretKey secretKey = CryptoUtils.deriveAESKey(privateKey, publicKey);
        return CryptoUtils.encryptAES_GCM(plaintext, secretKey); // Encrypt and return
    }

    /**
     * Decrypt data using AES-GCM.
     * @param encryptedData The encrypted data (Base64 encoded).
     * @param publicKeyEncoded The public key of the other party (Base64 encoded).
     * @param privateKeyEncoded Your private key (Base64 encoded).
     * @return Decrypted plaintext.
     * @throws Exception if decryption fails.
     */
    @PostMapping("/decrypt")
    public String decrypt(@RequestBody String encryptedData, @RequestParam String publicKeyEncoded, 
                          @RequestParam String privateKeyEncoded) throws Exception {
        // Decode Base64 keys
        PrivateKey privateKey = KeyFactory.getInstance("EC", "BC")
                .generatePrivate(new PKCS8EncodedKeySpec(Base64.decode(privateKeyEncoded)));
        PublicKey publicKey = KeyFactory.getInstance("EC", "BC")
                .generatePublic(new X509EncodedKeySpec(Base64.decode(publicKeyEncoded)));
        
        // Derive AES key
        SecretKey secretKey = CryptoUtils.deriveAESKey(privateKey, publicKey);
        return CryptoUtils.decryptAES_GCM(encryptedData, secretKey); // Decrypt and return
    }
}
```