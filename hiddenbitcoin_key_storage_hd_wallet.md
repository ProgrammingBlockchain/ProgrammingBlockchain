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

## Key storage design decisions  

Now it is a great time to give you a template on what decisions have to be make for storing the keys and what to keep in mind while making them.  

### Using only one key  

This is a shortcut. There are not too many situations I can come up with where going down this road can be justified. Yet it is not impossible it will be the most suitable for your needs.  
As a bad example here is an illustration for a Bitcoin wallet I have built, what only uses one key. I leave it to you to think of the consequences.  

![](assets/TransparentWallet2.png)  

### JBOK wallets  

It stands for "Just a Bunch Of Keys". At the time of writing the reference client uses this method to store keys.  
The problem with this the user has to periodically backup his wallet. Yet if you want to be able importing or dropping keys, changing password you need to use this or some kind of hybrid combination of this and a deterministic wallet. I decided not to use this since my HiddenWallet is trying to innovate towards privacy and I can have a more sound wallet without it.  

### HD wallets  
