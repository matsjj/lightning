# Decoding TCP Frames

## Status

Draft

## Author

Mats Jerratsch, Blockchain
mats@blockchain.com

## Abstract

Each message is prepadded with 4 bytes encoding the amount of bytes of the message. That length itself is excluded in that amount.

Right now the length of a message is not encrypted. This will likely be adapted in the future to mitigate traffic analysis.