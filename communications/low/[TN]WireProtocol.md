# Wire Protocol

## Status

Draft

## Author

Mats Jerratsch, Blockchain
mats@blockchain.com

## Abstract

Nodes that participate within the Lightning Network are communicating to create and settle payment channels and send, redeem and refund payments. 

There are different kind of messages, based on the purpose and the state of the communications.

All messages are serialised using JSON. 

# Specifications

For easier prototyping, all messages are serialised using JSON. This increases the size of the data by a factor of ~300%, if this becomes a problem it is easy to adapt a more efficient protocol (likely Protobufs).

The type of the message is prepadded into the string as an additional field

```
{"type": "EncryptionMessage", "data": "..."}
```

Each node MUST fail the connection if a message does not include a necessary field.

See the specification of the actual layer for the respective implementation. 
