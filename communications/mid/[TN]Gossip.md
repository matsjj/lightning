# Gossip Broadcasting

## Status

Draft

## Author

Mats Jerratsch, Blockchain
mats@blockchain.com

## Abstract

There is various meta data about the topology of the Lightning Network to be spread across the network. We use a gossip protocol similar to the one in bitcoin.

## Specification

Whenever a node determines that it should update the network about its current status, it will create a `P2PDataObject` to hold this information and tries to broadcast this to the network.

To safe bandwidth, broadcasting updates consists of three phases, with one peer sending a list of new inventory item hashes, the other peer requesting unknown messages and then sending the requested objects.

### GossipInvMessage

```
    public List<byte[]> inventoryList;
```

When a certain threshold is reached, a node will send a list of hashes of new objects. 

### GossipGetMessage

```
    public List<byte[]> inventoryList;
```

Upon receiving a `GossipInvMessage`, a node will go through its database to see which of these objects are still unknown. It will then send a `GossipGetMessage` to request unknown objects. If it knows all objects already, the node MAY choose to ignore `GossipInvMessage`.


### GossipSendMessage

```
    public List<PubkeyIPObject> pubkeyIPList;
    public List<PubkeyChannelObject> pubkeyChannelList;
    public List<ChannelStatusObject> channelStatusList;
```

Upon receiving a `GossipGetMessage`, a node will construct a `GossipSendMessage`, including all requested objects. 



## P2PDataObject

Currently there are three defined objects for spreading meta data. Each object MUST define a way to construct the hash and it MUST include a timestamp of creation.
Furthermore, all objects MUST include the global public key of the node, who initially broadcasted the object and a valid signature signing the hash of the object.
### PubkeyChannelObject

To mitigate the risk of MITM attacks against new nodes we check all channels against the blockchain. This way an attacker would need to lock up and spend a lot of funds on the blockchain before even able to try attacking a new node.

As with the current design the anchor transactions pay to P2SH, it is impossible to rebuild the output without the keys. Therefore we have to send the keys and anchors together with the txid. We further do not use the public node key in the anchor but rather prepend it and sign with the key from the anchor to not have to reuse the node key for each anchor. [TODO: Both nodes communicate to sign the object in order to save them from both sending an object about the same channel.]

The current anchor design pays to

```
    OP_HASH <SECRET-A-HASH> OP_EQUAL
    OP_IF
    <KEY-B’>
    OP_ELSE
    <KEY-B>
    OP_ENDIF
    2 OP_SWAP
    <KEY-A> 2 OP_CHECKMULTISIG
```

so for other people to be able to decrypt the P2SH we need to distribute

```
    public byte[] secretAHash;
    public byte[] secretBHash;
    public byte[] pubkeyB;
    public byte[] pubkeyB1;
    public byte[] pubkeyB2;
    public byte[] pubkeyA;
    public byte[] pubkeyA1;
    public byte[] pubkeyA2;
    public byte[] txidAnchor;
```

### PubkeyIPObject

To help sustain connections even with dynamic IPs and help new nodes connect to other nodes, we distribute the IP addresses of those nodes that wish to broadcast it. 

```
    public String hostname;
    public int port;
```

### Relationship Channel - Status

To help make educated decisions on routing, each node does broadcast the status of a channel at different intervals. This mainly depends on the activity on the channel, especially channels that receive lots of payments in one direction within a short time. The status object will include data about capacity and fees for receiving payments.

For now, routing based on fees / capacity is not yet implemented, and nodes don't broadcast updates yet based on the status of the channel. They are used to allow routing in general though.

```
    public byte[] pubkeyA;
    public byte[] pubkeyB;
    public byte[] infoA;
    public byte[] infoB;
    public int latency;
    public int feeA;
    public int feeB;
```