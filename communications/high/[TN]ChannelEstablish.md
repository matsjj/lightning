# Channel Establishment

## Status

Draft

## Author

Mats Jerratsch, Blockchain
mats@blockchain.com

## Abstract

Payments in the Lightning Network are done by shifting balances from one party to another in so called payment channels. The construction of the payment channel is done by sending back and forth the exact details of the channel and signatures that allow to refund the channel committed funds if anything were to fail.

## Specification

Currently we are exchanging 4 types of message:

```
       Alice           Bob
 
         A     ->
               <-       A'
         B     ->
               <-       B'
         C     ->
               <-       C'
         D     ->
               <-       D'
```

### Phase A (channel establishment messages)

The node that connects is usually the node that wants to create a payment channel with the other node, we call this node Alice, and its counterparty Bob. Upon connection, Alice will send a `LNEstablishAMessage`, containing:

```java
// Data about the anchor
public byte[] channelKeyServer;       // Public key (Alice)
public long amountClient;             // Amount Bob should put in channel
public long amountServer;             // Amount Alice commits to channel
public byte[] anchorTransaction;      // Anchor with inputs from Alice
public byte[] addressBytes;           // Bitcoin address from Alice
public int minConfirmationAnchor;     // Minimum anchor confirmations

// Data about the first commitment to be able to refund if necessary
public RevocationHash revocationHash; // Revocation hash from Alice
public int feePerByte;                // On-chain transaction fee
public long csvDelay;                 // Revocation grace period
```

See below for more information about the final anchor transaction.

Alice will send `amountServer`, the amount it will put into the channel, and `amountClient`, the amount the node wishes the other node contributes into the channel. Currently all amounts are in satoshi. This will get changed to millisatoshi in a later release.

Upon receipt of an `LNEstablishAMessage`, Bob will reply with another `LNEstablishAMessage`, (optionally) appending inputs and change outputs to the anchor, but still not signing it. If Bob is not required to put funds in the channel, then it is not necessary for him to alter the anchor transaction.

Both parties MUST check all inputs of `anchorTransaction` to make sure that each input is paying to a witness-formatted scriptPubKey, such that the final anchorTransaction is not subject to transaction malleability.

### Phase B (commitment transaction signatures)

Once alice gets Bob’s A message, she can send her commitment signature. Technically Bob does not need to wait for Alice to do the same, but in the current implementation it is still the case that all phase A messages must have been exchanged before phase B messages are.

```java
public byte[] channelSignature;       // Signature for contparty’s commitment
```

### Phase C (anchor signatures)

Upon receipt and verification of commitment signature in message B, message C is sent containing the anchor transaction with all our signatures.

```java
public byte[] anchorSigned;             // Anchor with counterparty signature(s)
```

### Phase D (channel established)

Once a node sees enough confirmations for the anchor, it sends its counterparty a D message that means that the channel has been established. The D message does not carry any data. Blockchain monitoring has not yet been implemented. Currently the library will not send these messages.

## Anchor Transaction

The final design of the anchor will look like

```
   In:
  txInA1
  txInA2
  […]
  txInB1
  txInB2
  […]
   Out:
  2-of-2 A & B
  Change A
  Change B
```

The 2-of-2 has keys lexicographically ordered. 