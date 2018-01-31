# Block explorers {#block-explorers}

A block explorer can get you started very quickly. You used `QBitNinja` already in this book, but many more exists.
A block explorer is a self-hosted or third-party hosted solution which provide you informations about blocks, transactions and addresses in the chain.

![Explorer](../assets/Wallet-Explorer.png)
A block explorer basically connect to a bitcoin node, index the data of the blockchain and expose an easy to use JSON API.
Solutions include: `QBitNinja`, `Blockcypher`, `Smartbit`, `Electrum server`, `Insight`, `NBXplorer`.

The advantages are:

* Better API than Bitcoin Core RPC
* Can handle more load
* Support large number of wallet, and can add them dynamically

The disadvantages are:

* If it is hosted by third party, and there is a contentious fork, you don't have the choice of which fork to follow
* Sometimes, their services are not enough to handle everything you need for a full wallet

Different block explorers expose quite different API and features. Block explorers never have to the keys of the wallet.

With `QBitNinja`, it is difficult to track a wallet which always change addresses, because you need to poll all the addresses belonging to the same wallet to detect any change.

However, `Electrum` or `NBxplorer` exposes notifications via websockets  or long polling so you don't need to poll all the addresses of the wallet.
`Insight` is not well maintained, and `Blockcypher` or `Smartbit` are third-party hosted.

[NBxplorer](https://github.com/dgarage/NBXplorer/) has been created to have a very simple API, is self hostable, and track only what is needed for your wallet.
Contrary to `QBitNinja`, it relies on you having a full node, but it provides websocket notifications and easy way to query the balances of a wallet.

NBXplorer is also multi crypto currency on a single server. (supporting Bitcoin and Litecoin at the time of the writing)
It has been thought to integrate nicely with `NBitcoin`.

You need a fully synched `bitcoind` node with default parameters then:
Clone and run [NBXplorer](https://github.com/dgarage/NBXplorer) with default parameters.

Reference the `NBXplorer.Client` nuget package then you first need to notify the `NBXplorer` to track the user wallet:

```
var network = new NBXplorerNetworkProvider(ChainType.Main).GetBTC();
var userExtKey = new ExtKey();
var userDerivationScheme = network.DerivationStrategyFactory.CreateDirectDerivationStrategy(userExtKey.Neuter(), new DerivationStrategyOptions()
{
	// Use non-segwit
	Legacy = true
});
ExplorerClient client = new ExplorerClient(network);
client.Track(userDerivationScheme);
```

Change `ChainType.Main` if you want to use Testnet or Regtest.

Then you can query the UTXOs of your user and spend them the following way:

```
var utxos = client.GetUTXOs(userDerivationScheme, null, false);
```

If you want to spend those UTXOs:

```
var coins = utxos.GetUnspentCoins();
var keys = utxos.GetKeys(userExtKey);
TransactionBuilder builder = new TransactionBuilder();
builder.AddCoins(coins);
builder.AddKeys(keys);
builder.Send(new Key(), Money.Coins(0.5m));
builder.SetChange(changeAddress.ScriptPubKey);

// Set the fee rate
var fallbackFeeRate = new FeeRate(Money.Satoshis(100), 1);
var feeRate = tester.Client.GetFeeRate(1, fallbackFeeRate).FeeRate;
builder.SendEstimatedFees(feeRate);
/////

var tx = builder.BuildTransaction(true);
Console.WriteLine(client.Broadcast(tx));
```

A problem with this solution is that if you call this code twice at the exact same time, you will likely broadcast two transactions spending the same coins, resulting in one of the transaction getting dropped.

To prevent this problem, you need to make sure to not spend twice the same coins.

A way to solve the problem is by simply retrying:

```
while(true)
{    
    var coins = utxos.GetUnspentCoins();
    var keys = utxos.GetKeys(userExtKey);
    TransactionBuilder builder = new TransactionBuilder();
    builder.AddCoins(coins);
    builder.AddKeys(keys);
    builder.Send(new Key(), Money.Coins(0.5m));
    builder.SetChange(changeAddress.ScriptPubKey);

    // Set the fee rate
    var fallbackFeeRate = new FeeRate(Money.Satoshis(100), 1);
    var feeRate = tester.Client.GetFeeRate(1, fallbackFeeRate).FeeRate;
    builder.SendEstimatedFees(feeRate);
    /////

    var tx = builder.BuildTransaction(true);
    var result = client.Broadcast(tx);
    if(result.Success)
    {
        Console.WriteLine("Success!");
        break;
    }
    else if(result.RPCCode.HasValue && result.RPCCode.Value == RPCErrorCode.RPC_TRANSACTION_REJECTED)
    {
        Console.WriteLine("We probably got a conflict, let's try again!");
        continue;
    }
    else
    {
        Console.WriteLine($"Something is really wrong {result.RPCCode} {result.RPCCodeMessage} {result.RPCMessage}");
        // Do something!!!
    }
}
```

Another common way is to have a global list of already used outpoint that you can check against.