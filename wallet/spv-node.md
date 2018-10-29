# SPV Node {#spv-node}

In order to enable decentralized light clients, SPV wallets were invented. Just like a full SPV wallet, SPV wallets do SPV validation, however SPV wallets are downloading only transactions, and not full blocks from Bitcoin nodes. Such simple client has not been implemented, because of its obvious privacy drawbacks. In order to solve these privacy problems Bloom filtering SPV wallets were introduced with [BIP37](https://github.com/bitcoin/bips/blob/master/bip-0037.mediawiki) and became popular. Later on bloom filtering SPV wallets [turned out to be](https://jonasnick.github.io/blog/2015/02/12/privacy-in-bitcoinj/) a privacy nightmare.  
An C# implementation sample with somewhat improved privacy can be found on GitHub: [NBitcoin.SPVSample](https://github.com/NicolasDorier/NBitcoin.SPVSample).  

A newer version of SPV wallets, called Compact Client Side Filtering improves upon the privacy of BIP37 with a different architecture. Such implementation, called [Neutrino](https://github.com/lightninglabs/neutrino) exists today.
