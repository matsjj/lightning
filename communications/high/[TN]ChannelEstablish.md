# Channel Establishment

## Status

Draft

## Author

Mats Jerratsch, Blockchain
mats@blockchain.com

## Abstract

Payments in the Lightning Network are done by shifting balances from one party to another in so called payment channels. The construction of the payment channel is done by sending back and forth the exact details of the channel and signatures that allow a safe revocation of any possible refund after the establish process is completed.

## Specification

Channel creation in this document follows the approach described in deployable-lightning[1]. It will soon be replaced by a less-complicated solution, utilizing the feature set from Segregated Witness.

Currently we are exchanging 4 messages,
```
       Alice           Bob
 
         A     ->
               <-       A'
               <-       B
         B'    ->
               <-       C
         C'    ->
```

### Phase A (channel establishment messages)

The node that connects is usually the node that wants to create a payment channel with the other node, we call this node Alice, and its counterparty Bob. Upon connection, Alice will send a `LNEstablishAMessage`, containing:

```
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

Alice will send `amountServer`, the amount it will put into the channel, and `amountClient`, the amount the node wishes the other node contributes into the channel. Currently all amounts are in satoshi. This will get changed to millisatoshi in a later release.

Upon receipt of an `LNEstablishAMessage`, Bob will reply with another `LNEstablishAMessage`, (optionally) appending inputs and change outputs to the anchor, but still not signing it. If Bob is not required to put funds in the channel, then it is not necessary for him to alter the anchor transaction.

### Phase B (commitment transaction signatures)

Once alice gets Bob’s A message, she can send her commitment signature. Technically Bob does not need to wait for Alice to do the same.

```
public byte[] channelSignature;       // Signature for contparty’s commitment
```

### Third phase (anchor signatures)

Upon receipt and verification of commitment signature in message B, message C is sent containing the anchor transaction with all our signatures.

```
public byte[] anchorSigned;             // Anchor with counterparty signature(s)
```

## Resources

[1] https://github.com/ElementsProject/lightning