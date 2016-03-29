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
               <-       B
         C     ->
               <-       D
```

The node that connects is usually the node that wants to create a payment channel with the other node. Upon connection, it will send a `LNEstablishAMessage`, containing

```
    public byte[] pubKeyEscape;
    public byte[] pubKeyFastEscape;
    public byte[] secretHashFastEscape;
    public byte[] revocationHash;
    public long clientAmount;
    public long serverAmount;
```

where `pubKeyEscape`,`pubKeyFastEscape`,`secretHashFastEscape` and `revocationHash` are all used to create the channel in a way that makes it possible to (1) refund if the other party does disappear before broadcasting their half of the anchor (2) revoke the described refund in (1) after fully establishing the channel [1].

The node will send `serverAmount`, the amount it will put into the channel, and `clientAmount`, the amount the node wishes the other node contributes into the channel. Currently all amounts are in satoshis, this will get changed to use millisatoshi in a later release.

Node B will respond with `LNEstablishBMessage`, containing

```
    public byte[] pubKeyEscape;
    public byte[] pubKeyFastEscape;
    public byte[] secretHashFastEscape;
    public byte[] revocationHash;
    public byte[] anchorHash;
    public long serverAmount;
```

with similar contents as in `LNEstablishAMessage`. Node B can adjust `serverAmount`, if it wishes to change the amount it puts into the channel. If either node is not contented with the amounts each node puts into the channel, they MUST close the connection. 
It also contains `anchorHash`, the hash of the anchor transaction of node B. 

Node A will respond with `LNEstablishCMessage`, containing

```
    public byte[] signatureEscape;
    public byte[] signatureFastEscape;
    public byte[] anchorHash;
```

with two signatures for escape transactions as outlined it [1]. The escape transactions point at the `anchorHash` sent in `LNEstablishBMessage`. Node B MUST confirm that the signatures are indeed correct or it is bound to lose all money put into the channel. 

Afterwards node B responds with `LNEstablishDMessage`, containing

```
    public byte[] signatureEscape;
    public byte[] signatureFastEscape;
```

similar as in `LNEstablishCMessage`. Node B MUST check the signatures. 

After receiving either `LNEstablishCMessage` or `LNEstablishDMessage`, each node will broadcast their anchor and wait for sufficient confirmations. Afterwards they are able to process payments. [TODO: Implement a channel_ready message]

## Resources

[1] https://github.com/ElementsProject/lightning