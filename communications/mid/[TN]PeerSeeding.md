# Peer Seeding

## Status

Draft

## Author

Mats Jerratsch, Blockchain
mats@blockchain.com

## Abstract

Due to the decentralised nature of the Lightning Network, there is no central point of connection for new nodes that just joined the network. Unfortunately, there is not yet a perfect solution to find other peers running a particular service over the whole network in a reliable way. 

In general, networks follow some sort of centralized way to solve this way. For example, in the early days of Bitcoin, nodes joined an IRC channel and changed their username to encode their IP. Eventually it converged to some statically hardcoded nodes that will send them a list of other nodes. TN will follow this approach.

## Specification

There are currently a few hardcoded nodes that can send back a fixed amount of nodes. Beside the IP/port, the public key of that node is stored as well, to mitigate MITM attacks against new nodes. To open for decentralisation, we should aim to add as many long-running stable nodes into the pool of seed nodes as possible in the future.

Upon requesting new peers, the new node will send `PeerSeedGetMessage`, to which the other node will send `PeerSeedSendMessage`, containing

```
    public List<PubkeyIPObject> ipObjectList;
```

which will contain a random collection of recent nodes.

If the other node is not answering the request within reasonable time, the node MAY close the connection.