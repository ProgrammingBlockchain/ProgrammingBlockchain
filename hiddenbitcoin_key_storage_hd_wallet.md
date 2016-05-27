# HiddenBitcoin: Key Storage (HD wallet)

([nopara73](https://github.com/nopara73)) I am developing a privacy oriented Bitcoin wallet, called [HiddenWallet](https://github.com/nopara73/HiddenWallet). The [HiddenBitcoin](https://github.com/nopara73/HiddenBitcoin) library is the introduction of an other abstraction layer between NBitcoin and the user interface.  

In practice we have no clear definition of what we mean by the word "wallet" and what it is supposed to do.  
For example we say [HD wallet](https://en.bitcoin.it/wiki/Deterministic_wallet), but a HD wallet does not check The Blockchain and send money, it is simply a method of storing keys. In the contrary we also call the [Electrum client](https://electrum.org/) a wallet and we expect it to be able to track our keys and send money.  
HiddenBitcoin identifies three key function a Bitcoin wallet does and this tutorial will be structured around them:  

1. Securely stores keys and manages the access to them.  
2. Monitors these keys, and other keys on The Blockchain.  
3. Builds transactions and submits them.  

In this lesson I am going to tackle the key storage function.  
If you want to examine the code more extensively you can find the solution on [GitHub](https://github.com/nopara73/HiddenBitcoin).  
If you just want to know how to quickly set it up and use it you can find my high level tutorial on [CodeProject](http://www.codeproject.com/Articles/1096320/HiddenBitcoin-High-level-Csharp-Bitcoin-wallet-lib).  

**How high level is it?** In my opinion a GUI developer, designer should not be able to make too much mistakes. They should not know about inputs and outputs and scriptpubkeys. They should stick at the addresses, privatekeys and wallets level. Also NBitcoin should be fully abstracted away.  

