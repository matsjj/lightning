# Gossip

## Outline

There is various meta data about the network like topology to be spread. We use a gossip protocol similar to the one in bitcoin.

## Specifics

### Meta Data

Currently there are three defined objects for spreading meta data

#### Relationship Node Key - Open Channels

To mitigate the risk of MITM attacks against new nodes we check all channels against the blockchain. This way an attacker would need to lock up and spend a lot of funds on the blockchain before even able to try attacking a new node.

As with the current design the anchor transactions pay to P2SH, it is impossible to rebuild the output without the keys. Therefore we have to send the keys and anchors together with the txid. We further do not use the public node key in the anchor but rather prepend it and sign with the key from the anchor to not have to reuse the node key for each anchor. Both nodes communicate to sign the object in order to save them from both sending an object about the same channel. 

The current anchor design pays to 

OP_HASH <SECRET-A-HASH> OP_EQUAL
OP_IF
<KEY-B’>
OP_ELSE
<KEY-B>
OP_ENDIF
2 OP_SWAP
<KEY-A> 2 OP_CHECKMULTISIG


so for other people to be able to decrypt the P2SH we need to distribute 

SECRET-A-HASH
SECRET-B-HASH
KEY-A
KEY-A’
KEY-B
KEY-B’

#### Relationship IP - Node Pubkey

To help sustain connections even with dynamic IPs and help new nodes connect to other nodes, we distribute the IP addresses of those nodes that wish to broadcast it. 

#### Relationship Channel - Status

To help make educated decisions on routing, each node does broadcast the status of a channel at different intervals. This mainly depends on the activity on the channel, especially channels that receive lots of payments in one direction within a short time. The status object will include data about capacity and fees for receiving payments.

### Gossip protocol

As the IP object is small and will occur reasonable seldom, we will send it without sending an inventory object first. If another node sends us an IP object we will check whether we have it in our database already and broadcast it further if we don’t. If we know it, we will just discard it.

For other objects we will send an inventory object containing hashes of data we got from other nodes recently. The other node then sends us a GET request with the hashes of the data it wants to receive. 
