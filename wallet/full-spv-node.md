# Full SPV Node {#full-spv-node}

Another, newer Bitcoin wallet architecture is the Full Block Downloading SPV wallet.  
Such wallets are [Jonas Schnelli's Bitcoin Core PR](https://github.com/bitcoin/bitcoin/pull/9483), nopara73's [HiddenWallet](https://github.com/nopara73/HiddenWallet) in C# and [Stratis's Breeze Wallet](https://github.com/stratisproject/Breeze) in C#.  

Compared to a full node, a full SPV node downloads all the blocks, but from the beginning of the wallet. This, however faster, it is not capable to fully validate blocks, it uses SPV validation instead.  
Its main advantage is compared to lighter wallet architectures: it provides full node level privacy against network observers.
