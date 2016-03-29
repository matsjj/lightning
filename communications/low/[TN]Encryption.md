# Encryption

## Status

Draft

## Author

Mats Jerratsch, Blockchain
mats@blockchain.com

## Abstract

At the beginning of each connection, both nodes will establish a shared secret for encrypting the rest of the communications.

To leak no information about the nodes, we will use only ephermal keys to establish the shared secret.

## Specification

### Handshake

After the connection was established, both nodes generate a secp256k1 keypair. They send the compressed 33byte public key to the other party. Both parties then calculate a common shared secret masterKey using the ECDH technique.

```
    public byte[] key;
```

Using the shared secret, they further derive further keys:

`encryptionKey = SHA256(masterKey || 0)`

`hmacKey = SHA256(masterKey || 1)`

`iv1 = SHA256(masterKey || pubkey1)`

`iv2 = SHA256(masterKey || pubkey2)`

With index 1 being our pubkey and communications in our directions, and index 2 being the pubkey of the other party and communications in their direction respectively.

### Encryption

We use AES-128-CTR to encrypt all following communications. The encrypted data is then authenticated using a HMAC technique with the derived key.

The last 8 byte of the initialisation vector `iv1` and `iv2` are overwritten with a counter of messages in the current direction.

The total packet will then be

```
    public byte[] hmac;
    public byte[] payload;
```
