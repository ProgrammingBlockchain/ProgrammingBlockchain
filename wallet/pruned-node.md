# Pruned Node {#pruned-node}

A pruned Bitcoin node is technically still a Bitcoin full node, however it prunes its local blockchain in order to save disk space. Both Stratis's Bitcoin node and Bitcoin Core is capable of pruning.  
Compared to an archeological full node, a pruned node cannot serve old blocks to other nodes, and it cannot use a few full node settings, most notably Bitcoin Core's `txindex=1`.  

Since those, who are looking at pruning compromise tend to want to lower other requirements of running a full node, too, therefore this is a good place to insert some ideas on how to do that in Bitcoin Core:  

```
prune=550

maxconnections=8
listen=0
maxuploadtarget=144

checkblocks=1
checklevel=0

txindex=0
```
