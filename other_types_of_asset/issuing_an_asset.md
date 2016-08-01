## Menerbitkan Aset {#issuing-an-asset}

### Penerbitan Koin {#issuance-coin}

Dalam Open Asset, ID aset berasal dari emiten **ScriptPubKey**.  
Jika anda ingin menerbitkan Colored Coin, anda perlu membuktikan kepemilikan **ScriptPubKey **tersebut. Dan satu-satunya cara untuk dapat melakukkan itu di Blockchain adalah dengan melakukan pengeluaran sejumlah koin pada **ScriptPubKey**.

Koin yang hendak dikeluarkan atau dibelanjakan untuk menerbitkan colored coins tersebut disebut dengan “**Issuance Coin**”. Di dalam **NBitcoin**.  
I want to emit an Asset from the book bitcoin address: [1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB](https://www.smartbit.com.au/address/1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB).

Take a look at my balance, I decided to use the following coin for issuing assets.

```json
{
          "transactionId": "eb49a599c749c82d824caf9dd69c4e359261d49bbb0b9d6dc18c59bc9214e43b",
          "index": 0,
          "value": 2000000,
          "scriptPubKey": "76a914c81e8e7b7ffca043b088a992795b15887c96159288ac",
          "redeemScript": null
} 
```

Here is how to create my issuance coin:

```cs
var coin = new Coin(
    fromTxHash: new uint256("eb49a599c749c82d824caf9dd69c4e359261d49bbb0b9d6dc18c59bc9214e43b"),
    fromOutputIndex: 0,
    amount: Money.Satoshis(2000000),
    scriptPubKey: new Script(Encoders.Hex.DecodeData("76a914c81e8e7b7ffca043b088a992795b15887c96159288ac")));

var issuance = new IssuanceCoin(coin);
```

Now I need to build transaction and sign the transaction with the help of the **TransactionBuilder**.

```cs
var nico = BitcoinAddress.Create("15sYbVpRh6dyWycZMwPdxJWD4xbfxReeHe");
var bookKey = new BitcoinSecret("???????");
TransactionBuilder builder = new TransactionBuilder();

var tx = builder
    .AddKeys(bookKey)
    .AddCoins(issuance)
    .IssueAsset(nico, new AssetMoney(issuance.AssetId, quantity: 10))
    .SendFees(Money.Coins(0.0001m))
    .SetChange(bookKey.GetAddress())
    .BuildTransaction(true);

Console.WriteLine(tx);
```

```json
{
  …
  "out": [
    {
      "value": "0.00000600",
      "scriptPubKey": "OP_DUP OP_HASH160 356facdac5f5bcae995d13e667bb5864fd1e7d59 OP_EQUALVERIFY OP_CHECKSIG"
    },
    {
      "value": "0.01989400",
      "scriptPubKey": "OP_DUP OP_HASH160 c81e8e7b7ffca043b088a992795b15887c961592 OP_EQUALVERIFY OP_CHECKSIG"
    },
    {
      "value": "0.00000000",
      "scriptPubKey": "OP_RETURN 4f410100010a00"
    }
  ]
}
```

You can see it includes an OP\_RETURN output. In fact, this is the location where information about colored coins are stuffed.

Here is the format of the data in the OP\_RETURN.

![](../assets/ColorMaker.png)

In our case, Quantities have only 10, which is the number of Asset I issued to `nico`. Metadata is arbitrary data. We will see that we can put an url that points to an “Asset Definition”.  
An **Asset Definition** is a document that describes what the Asset is. It is optional, we are not using it in our case. \(We’ll come back later on it in the Ricardian Contract part.\)

For more information check out the [Open Asset Specification](https://github.com/OpenAssets/open-assets-protocol/blob/master/specification.mediawiki).

After transaction verifications it is ready to be sent to the network.

```cs
Console.WriteLine(builder.Verify(tx)); 
```

### Dengan QBitNinja

```cs
var client = new QBitNinjaClient(Network.Main);
BroadcastResponse broadcastResponse = client.Broadcast(tx).Result;

if (!broadcastResponse.Success)
{
    Console.WriteLine("ErrorCode: " + broadcastResponse.Error.ErrorCode);
    Console.WriteLine("Error message: " + broadcastResponse.Error.Reason);
}
else
{
    Console.WriteLine("Success!");
}
```

### Atau dengan Bitcoin core

```cs
using (var node = Node.ConnectToLocal(Network.Main)) //Connect to the node
{
    node.VersionHandshake(); //Say hello
    //Advertize your transaction (send just the hash)
    node.SendMessage(new InvPayload(InventoryType.MSG_TX, tx.GetHash()));
    //Send it
    node.SendMessage(new TxPayload(tx));
    Thread.Sleep(500); //Wait a bit
}
```

My Bitcoin Wallet have both, the book address and the “Nico” address.

![](../assets/NicoWallet.png)

As you can see, Bitcoin Core only shows the 0.0001 BTC of fees I paid, and ignore the 600 Satoshi coin because of spam prevention feature.

This classical bitcoin wallet knows nothing about Colored Coins.  
Worse: If a classical bitcoin wallet spend a colored coin, it will destroy the underlying asset and transfer only the bitcoin value of the **TxOut**. \(600 satoshi\)

For preventing a user from sending Colored Coin to a wallet that do not support it, Open Asset have its own address format, that only colored coin wallets understand.

```cs
nico = BitcoinAddress.Create("15sYbVpRh6dyWycZMwPdxJWD4xbfxReeHe");
Console.WriteLine(nico.ToColoredAddress());
```

```
akFqRqfdmAaXfPDmvQZVpcAQnQZmqrx4gcZ
```

Now, you can take a look on an Open Asset compatible wallet like Coinprism, and see my asset correctly detected:

![](../assets/Coinprism.png)

As I have told you before, the Asset ID is derived from the issuer’s **ScriptPubKey**, here is how to get it in code:

```cs
var book = BitcoinAddress.Create("1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB");
var assetId = new AssetId(book).GetWif(Network.Main);
Console.WriteLine(assetId); // AVAVfLSb1KZf9tJzrUVpktjxKUXGxUTD4e
```

