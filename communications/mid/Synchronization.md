# Synchronization

## Outline

When a new node joins the network, there is some meta-data it wants to collect before establishing actual payment channels to other nodes, as this is accosiated with real costs.

## Specifics

There are currently three different types of meta-data and they all will get downloaded in this step. See Gossip for a detailed explanation of the different objects.

There are multiple steps involved in getting the topology of the network.

### Step 1 - Peer Discovery

We are querying some known nodes for peer discovery. We only rely on these nodes for obtaining a list of random IP addresses of other nodes of the network. 

After obtaining said list, we will disconnect from that node again to not rely on a central node too much.

### Step 2 - Download Meta-Data

The meta-data is indexed using a simple hash function and then partitioned into 1000 segments, based on the hash.
Our node will connect to random nodes and query for a specific segment index. It will return to us with all objects that lie in the hash-region associated with said index.

This way we can distribute the load of downloading the complete data set and also mitigate some risks of dishonest participants. If necessary, it further allows us to check specific segments with many different nodes.

### Step 3 - Validate Data

As part of the data is reflected on the blockchain, we can use it to make sure that said data is correct. 
Data that we are unable to validate will be discarded.

As we can validate the meta data on the blockchain, it is more difficult for an attacker to present us with a wrong set of data. In case an attacker wants to push us onto a fake network that he controls, he would need to commit a lot of funds into actual bitcoin transactions.

### Step 4 - Open Lightning Channels

Now that we have a good representation of the network topology, we can estimate with which nodes we want to have channels with.
We can use different algorithms and weighting to make sure we end up making
- the network more robust against clustering
- good use of the capital we used to open the channel.

In a real world there will be a trade-off between lots of connections in a local area (most payments happen locally) and connections to nodes far across the network.

