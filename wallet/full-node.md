# Full Node {#full-node}

This is the oldest and the most recommended solution.  
You may use [Bitcoin Core's RPC](https://bitcoin.org/en/developer-reference#remote-procedure-calls-rpcs) through NBitcoin or the [C# Bitcoin Full Node](https://github.com/stratisproject/StratisBitcoinFullNode), created by Stratis.   
Its pros are trustlessness and high network level privacy, while its cons are high storage, bandwidth, CPU and time requirements.   
In the rest of this document we will discuss Bitcoin Core's RPC API. First you need to fully syncronize your Bitcoin Core node, then you can do all operations above by calling RPC commands on your Bitcoin Core instance.  

![Bitcoin Core](../assets/Wallet-Bitcoin-Core.png)

After Initial Blockchain Downloading (IBD,) which can take days, start `bitcoind`. Then use NBitcoin's `RPCClient` class to manage your wallet.  

```
RPCClient client = new RPCClient(Network.Main);
Console.WriteLine(client.GetNewAddress()); # Generate a new address
Console.WriteLine(client.GetBalance()); # Get the balance
```

The advantages are:

* Widely tested software,
* In case of contentious fork, you can decide by yourself which fork to follow,
* Good documentation

Disadvantages are:

* API sometimes difficult to use and cumbersome,
* You need a fully synced node, where IBD takes days,
* Bitcoin Core may not be sufficient for wallet with too many transactions,
* Need to restart `bitcoind` for adding additional wallet,
* Supports limited number of wallets. (Less than 10?)

I advise this solution if you have low volume of transactions, and your system does not require you to create wallets dynamically.  
Older versions of Bitcoin Core supported wallet accounts. While this is not the case anymore, you can either implement accounts by yourself on top of RPC or use [Bitcoin Knots](https://github.com/bitcoinknots/bitcoin), which is luke-jr's Bitcoin Core fork, where he is still maintaining the account functionalities.
