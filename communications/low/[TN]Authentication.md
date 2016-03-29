# Authentication

## Status

Draft

## Author

Mats Jerratsch, Blockchain
mats@blockchain.com

## Abstract

Nodes within the Lightning Network will make connections over unsafe mediums. To prevent MITM attacks, all communication start with an authentication handshake.

## Specifications

The node that has started the connection MUST send a `AuthenticationMessage` message first. The other node then checks the message for validity and MUST response with its own `AuthenticationMessage`.

If the nodes receives any other message than a `AuthenticationMessage` before this handshake, it MUST fail this connection. It MAY also fail the connection if it does not receive an `AuthenticationMessage` within a reasonable timeframe.

The `AuthenticationMessage` consists of

```
    public byte[] pubKeyServer;
    public byte[] signature;
```

where `pubKeyServer` is the public key of the node that is sending this message, and `signature` is a signature of 

```
pubKeyServer || ephemeralPubKeyClient
```

Each node MUST check the validity of the signature, and it MUST fail if it does not match with the `pubKeyServer`.

Furthermore, the node that opened the connection MUST check if the other node sent back the expected `pubKeyServer`, and fail the connection otherwise.