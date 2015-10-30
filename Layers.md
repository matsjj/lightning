#Layers

For sufficient abstraction and separation, we look at the different network layers independently.

##Low-Level 

### Frame Decoding

We prepend each message with 4 bytes representing the length of the frame. This is due to TCP fragmentation. We remove these 4 bytes and give the full message - once received - to the next layer.

### Encryption

The bytestream of the complete message is encrypted using AES-128-CTR. This layer will give a decrypted byte array to the next layer.

### Message Serialization

There are different techniques currently in use to convert the byte array to an actual message object. These vary in efficiency (Protobufs) and readability(JSON) useful for debugging. 

##Mid-Level

### Authentication

Both nodes authenticate with their public node key to the other node by signing some message with the private key.

### Syncronisation

New nodes want to download the whole dataset from the network. 

### Gossip

Nodes will broadcast various data using the P2P network. 

##High-Level

### Lightning Channel Establishment

For interacting on the Bitcoin Lightning Network channels will have to have payment channels with each other. 

### Channel Negotiation

Making payments, settling payments, working out the current state and balances of the channel, closing the channel again.








