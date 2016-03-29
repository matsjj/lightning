# Synchronisation

## Status

Draft

## Author

Mats Jerratsch, Blockchain
mats@blockchain.com

## Abstract

When a new node joins the network, there is some meta-data it wants to collect before establishing actual payment channels to other nodes, as this is accosiated with real costs.

## Specification

There are currently three different types of meta-data and they all will get downloaded in this step. See Gossip for a detailed explanation of the different objects.


### Download Meta-Data

The meta-data is indexed using a simple hash function and then partitioned into 1000 segments, based on the hash.
The node will connect to random nodes and query for a specific segment index. It will return to us with all objects that lie in the hash-region associated with said index.

This way it is possible to distribute the load of downloading the complete data set and also mitigate some risks of dishonest participants. If necessary, it further allows us to check specific segments with many different nodes.

To request a segment, the node will send `SyncGetMessage`, containing

```
    public int fragmentIndex;
```


The other node will then respond with `SyncSendMessage`, containing
```
    public List<PubkeyIPObject> pubkeyIPList;
    public List<PubkeyChannelObject> pubkeyChannelList;
    public List<ChannelStatusObject> channelStatusList;
```



### Validate Data

As part of the data is reflected on the blockchain, we can use it to make sure that said data is correct. 
Data that we are unable to validate will be discarded.

As we can validate the meta data on the blockchain, it is more difficult for an attacker to present us with a wrong set of data. In case an attacker wants to push us onto a fake network that he controls, he would need to commit a lot of funds into actual bitcoin transactions.

It is furthermore possible to analyse the data investigating the health of the network, checking for near-partitions and building up channels that might help the network.