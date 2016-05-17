# Payment Negotiation

## Status

Draft

## Author

Mats Jerratsch, Blockchain
mats@blockchain.com

## Abstract

Payments in the Lightning Network are done by shifting balances from one party to another in so called payment channels. When one party wants to make a new payment, both parties will create and sign a new version of the commitment transaction and provide revocation parameters to the counterparty.

## Specification

There are six messages exchanged when a payment is sent through the channel. This involves adding, redeeming or refunding arbitrary payments.

```
       Alice           Bob
 
         A     ->
               <-       B
         C     ->
               <-       C'
         D     ->
               <-       D'
```

Both, `LNPaymentCMessage` and `LNPaymentDMessage` are symmetrical, as both parties have to go through the same process of creating, validating and revoking new transactions and signatures.

The node that wants to change the current channel balance sends `LNPaymentAMessage`, containing:

```java
public int dice;                               // Random
public ChannelUpdate channelStatus;
public RevocationHash newRevocation;
```

`ChannelStatus`:

```java
public List<PaymentData> newPayments;
public List<PaymentData> refundedPayments;
public List<PaymentData> redeemedPayments;

public int feePerByte;
public long csvDelay;
```

`PaymentData`:

```java
public boolean sending;
public long amount;
public long fee;

public PaymentSecret secret;
public int timestampOpen;
public int timestampRefund;
public int csvDelay;

public OnionObject onionObject;
```

`PaymentSecret` and

```java
public byte[] secret;
public byte[] hash;
```

`OnionObject`

```java
public byte[] data;
```

Each node sends `dice` to resolve conflicts when both parties start the exchange at the same time. The node that sent a higher dice value will have priority for this exchange, while the other party has to start over afterwards.

Node B MUST make sure that all changed payments (new, redeemed, refunded) were previously part of the channel. Furthermore, node B MUST correctly calculate the resulting balances of applying these changes to the current status and fail the exchange if they don't match with the received amounts.

Node B MUST further check that `feePerByte`, `csvDelay` in each `PaymentData` and in the `ChannelStatus` is within the allowed boundaries.

Node B then responds with `LNPaymentBMessage` 

```java
public RevocationHash newRevocation;
```

Both nodes create the commit transaction for the other node plus a payment redeem/refund transaction for each payment. They will calculate the signatures for each of these and send it to the other node.

The commit transaction has the following structure

```
INPUT:
        ANCHOR 1
            sig A && sig B
        ANCHOR 2
            sig A && sig B
        ...
OUTPUT:
        AMOUNT_CLIENT to CSV_ENCUMBERED_CLIENT
        AMOUNT_SERVER to P2PKH_SERVER
        for each Payment
            if Payment is receiving
                AMOUNT_PAYMENT to 
                    (sign A && sign B && hash R) || (sign B && CLTV 5d) || (sign B || hash H)
            else
                AMOUNT_PAYMENT to 
                    (sign A && sign B && CLTV 5d) || (sign B && hash R) || (sign B || hash H)
                            
```

and the payment transaction has following structure

```
INPUT:
        COMMIT_TX
            if Payment is receiving
                sig A && sig B && hash R
            else
                sig A && sig B
OUTPUT:
        AMOUNT_PAYMENT to 
            (sign A && CSV 30d) || (sign B && hash H)
        
```

where `R` denominates the secret needed to redeem a payment, and `H` is the preimage of the revocation hash.

Node A and afterwards node B respond then with `LNPaymentCMessage`

```java
public byte[] newCommitSignature1;
public byte[] newCommitSignature2;
public List<byte[]> newPaymentSignatures;
```

where each node MUST check the validity of each signature.

Afterwards, both nodes exchange `LNPaymentDMessage`

```java
public List<RevocationHash> oldRevocationHashes;
```

Each node MUST check if it received all old revocation hashes and if they are indeed correct. 
