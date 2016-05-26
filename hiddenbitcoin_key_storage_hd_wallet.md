# HiddenBitcoin: Key Storage (HD wallet)

([nopara73](https://github.com/nopara73)) I am developing a privacy oriented Bitcoin wallet, called [HiddenWallet](https://github.com/nopara73/HiddenWallet). The [HiddenBitcoin](https://github.com/nopara73/HiddenBitcoin) library is the introduction of an other abstraction layer between NBitcoin and my user interface.  

In practice we have no clear definition of what we mean by the word "wallet" and what it supposed to do.  
For example we say [HD wallet](https://en.bitcoin.it/wiki/Deterministic_wallet), but a HD wallet does not monitors The Blockchain and send money, it is simply a method of storing keys. In the contrary we also call the [Electrum client](https://electrum.org/) to a wallet and we expect it to be able to monitor our keys and send money.