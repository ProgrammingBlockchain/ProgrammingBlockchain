# Bitcoin Core Wallet {#bitcoin-core-wallet}

This is the oldest and the most recommended solution.
You need a full synchronized Bitcoin Core node, then you can do all operations above by calling RPC commands on your Bitcoin Core instance.

![Bitcoin Core](../assets/Wallet-Bitcoin-Core.png)

You download bitcoin core, starts `bitcoind` then wait it synchronize (can take days).
You can then easily query and manage your wallet with the `RPCClient` class.

```
RPCClient client = new RPCClient(Network.Main);
Console.WriteLine(client.GetNewAddress()); # Generate a new address
Console.WriteLine(client.GetBalance()); # Get the balance
```

The advantages are:

* Widely tested software,
* In case of contentious fork, you can decide by youself which fork to follow,
* Good documentation

Disadvantages are:

* API sometimes difficult to use and cumbersome,
* You need a fully synched node, which take days,
* Bitcoin Core is not adapted for wallet with too much transactions
* Need to restart bitcoind for adding additional wallet
* Support limited number of wallets (less than 10?)

I advise this solution if you have low volume of transaction, and your system does not require you to create wallets dynamically.