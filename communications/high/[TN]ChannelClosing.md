# Channel Closing

## Status

Draft

## Author

Mats Jerratsch, Blockchain
mats@blockchain.com

## Abstract

Payments in the Lightning Network are done by shifting balances from one party to another in so called payment channels. If one party wants to redeem all of these payments in a single Bitcoin payment, he can ask to close the channel in collaboration with the other party.

## Specification

As we want to create a simple transaction, paying to (at most) 2 outputs, any still open payments needs to be dropped from the channel. They will be credited to the party who doesn’t necessarily wants to close the channel. If the node does not want to give up open payments, it must wait for them to settle or refund naturally and only then close the channel. It MUST NOT accept any new payments in this time span.

The node that wants to close the channel will need to calculate the correct amount for both parties. It MUST add the amount of all payments to the amount of the other party. It then has to create a transaction:

```
IN:     
        ANCHOR1
        ANCHOR2
OUT:    
        AMOUNT_CLIENT to P2PKH CLIENT
        AMOUNT_SERVER to P2PKH SERVER
                        
```

Where inputs and outputs has to be sorted as described in BIP69[1]. It has to choose a reasonable fee per KiB and deduct it evenly from both amounts. If either amount falls under the dust-limit, the node MUST drop the corresponding output completely and add it to the amount of the other party.

It then sends `LNCloseAMessage` to the other peer

```java
public List<byte[]> signatureList;
public float feePerByte;
```

where `signatureList` is a list of the signatures of all anchors. If `feePerByte` is out of bounds of the allowed boundaries, the receiving node MUST send back a new `LNCloseAMessage` with correct signatures and `feePerByte` set to an allowed value.

Upon receiving, the other node MUST validate the correctness of the signatures, send back correct signatures and broadcast the transaction to the Bitcoin network.

Both nodes MUST NOT accept OR send any payment messages, as it may lead to loss of funds. 

## Resources

[1] https://github.com/bitcoinjs/bip69