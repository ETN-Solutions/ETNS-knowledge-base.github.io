---
layout: home
parent: Encryption
title: GO lang Implementation
---


```go
package main

import (
	"crypto/aes"
	"crypto/cipher"
	"crypto/ecdsa"
	"crypto/rand"
	"crypto/x509"
	"encoding/base64"
	"encoding/pem"
	"fmt"
	"math/big"
)

// GenerateECDHKeyPair generates a new ECDH key pair using the P-256 curve.
func GenerateECDHKeyPair() (*ecdsa.PrivateKey, error) {
	priv, err := ecdsa.GenerateKey(elliptic.P256(), rand.Reader)
	if err != nil {
		return nil, err
	}
	return priv, nil
}

// DeriveAESKey derives an AES key from the shared secret.
func DeriveAESKey(privateKey *ecdsa.PrivateKey, publicKey *ecdsa.PublicKey) ([]byte, error) {
	// Calculate the shared secret
	x, _ := publicKey.Curve.ScalarMult(publicKey.X, publicKey.Y, privateKey.D.Bytes())
	return x.Bytes(), nil
}

// EncryptAESGCM encrypts plaintext using AES-GCM.
func EncryptAESGCM(plaintext string, aesKey []byte) (string, error) {
	block, err := aes.NewCipher(aesKey[:32]) // Use 256 bits for AES-256
	if err != nil {
		return "", err
	}

	// Generate a random 12-byte IV
	iv := make([]byte, 12)
	if _, err := rand.Read(iv); err != nil {
		return "", err
	}

	gcm, err := cipher.NewGCM(block)
	if err != nil {
		return "", err
	}

	ciphertext := gcm.Seal(nil, iv, []byte(plaintext), nil)

	// Combine IV and ciphertext
	result := append(iv, ciphertext...)
	return base64.StdEncoding.EncodeToString(result), nil
}

// DecryptAESGCM decrypts ciphertext using AES-GCM.
func DecryptAESGCM(encryptedData string, aesKey []byte) (string, error) {
	data, err := base64.StdEncoding.DecodeString(encryptedData)
	if err != nil {
		return "", err
	}

	// Extract IV and ciphertext
	iv := data[:12]
	ciphertext := data[12:]

	block, err := aes.NewCipher(aesKey[:32]) // Use 256 bits for AES-256
	if err != nil {
		return "", err
	}

	gcm, err := cipher.NewGCM(block)
	if err != nil {
		return "", err
	}

	plaintext, err := gcm.Open(nil, iv, ciphertext, nil)
	if err != nil {
		return "", err
	}

	return string(plaintext), nil
}

// EncodePublicKey encodes an ECDSA public key to PEM format.
func EncodePublicKey(pub *ecdsa.PublicKey) (string, error) {
	pubASN1, err := x509.MarshalPKIXPublicKey(pub)
	if err != nil {
		return "", err
	}
	block := &pem.Block{
		Type:  "PUBLIC KEY",
		Bytes: pubASN1,
	}
	return string(pem.EncodeToMemory(block)), nil
}

// EncodePrivateKey encodes an ECDSA private key to PEM format.
func EncodePrivateKey(priv *ecdsa.PrivateKey) (string, error) {
	privASN1, err := x509.MarshalECPrivateKey(priv)
	if err != nil {
		return "", err
	}
	block := &pem.Block{
		Type:  "EC PRIVATE KEY",
		Bytes: privASN1,
	}
	return string(pem.EncodeToMemory(block)), nil
}

func main() {
	// Generate ECDH key pair
	privKey, err := GenerateECDHKeyPair()
	if err != nil {
		fmt.Println("Error generating ECDH key pair:", err)
		return
	}

	// Encode keys to PEM format for display
	privPEM, _ := EncodePrivateKey(privKey)
	pubPEM, _ := EncodePublicKey(&privKey.PublicKey)

	fmt.Println("Private Key:\n", privPEM)
	fmt.Println("Public Key:\n", pubPEM)

	// Example usage for deriving AES key and encrypting
	// In a real application, you would exchange public keys with another party
	aesKey, err := DeriveAESKey(privKey, &privKey.PublicKey) // For demonstration, using own public key
	if err != nil {
		fmt.Println("Error deriving AES key:", err)
		return
	}

	// Encrypt a message
	plaintext := "Hello, this is a secret message!"
	encrypted, err := EncryptAESGCM(plaintext, aesKey)
	if err != nil {
		fmt.Println("Error encrypting:", err)
		return
	}
	fmt.Println("Encrypted:", encrypted)

	// Decrypt the message
	decrypted, err := DecryptAESGCM(encrypted, aesKey)
	if err != nil {
		fmt.Println("Error decrypting:", err)
		return
	}
	fmt.Println("Decrypted:", decrypted)
}

```