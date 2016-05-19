## Issuing an Asset {#issuing-an-asset}

### Objective {#objective}

For the purpose of this exercise, I will emit **BlockchainProgramming coins**.  

You get **one of these BlockchainProgramming coins** for every **0.004 bitcoin** you send me.  
**One more**  if you add some kind words.  
Furthermore this is a great opportunity to make it to the [Hall of The Makers](http://n.bitcoin.ninja/). 

Let’s see how I would code such feature.

### Issuance Coin {#issuance-coin}

In Open Asset, the Asset ID is derived from the issuer's **ScriptPubKey**.  
If you want to issue a Colored Coin, you need to prove ownership of such **ScriptPubKey**. And the only way to do that on the Blockchain is by spending a coin belonging to such **ScriptPubKey**.

The coin that you will choose to spend for issuing colored coins is called “**Issuance Coin**” in **NBitcoin**.  
I want to emit an Asset from the book bitcoin address: **1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB**.

By taking a look at [my balance](http://rapidbase-test.azurewebsites.net/balances/1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB?unspentOnly=true), I decided to use the following coin for issuing assets.

{

"transactionId": "eb49a599c749c82d824caf9dd69c4e359261d49bbb0b9d6dc18c59bc9214e43b",

"index": 0,

"value": 2000000,

"scriptPubKey": "76a914c81e8e7b7ffca043b088a992795b15887c96159288ac",

"redeemScript": null

}

Here is how to create my issuance coin.

Now I need to build transaction and sign the transaction with the help of the **TransactionBuilder**.

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

You can see it include an OP_RETURN output. In fact, this is the location where information about colored coins are stuffed.

Here is the format of the data in the OP_RETURN.

In our case, Quantities have only 10, which is the number of Asset I issued to nico. Metadata is arbitrary data. We will see that we can put a url that point to an “Asset Definition”.An “**Asset Definition**” is a document that describe what the Asset is. It is optional, we are not using it in our case. (We’ll come back later on it in the Ricardian Contract part)

For more information check out the [Open Asset Specification](https://github.com/OpenAssets/open-assets-protocol/blob/master/specification.mediawiki).

The transaction is ready to be sent on the network:

My Bitcoin Wallet have both, the book address and the “Nico” address.

As you can see, Bitcoin Core only show the 0.0001 BTC of fees I paid, and ignore the 600 Satoshi coin because of spam prevention feature.

This classical bitcoin wallet knows nothing about Colored Coins.Worse: If a classical bitcoin wallet spend a colored coin, it will destroy the underlying asset and transfer only the bitcoin value of the **TxOut**. (600 satoshi)

For preventing a user from sending Colored Coin to a wallet that do not support it, Open Asset have its own address format, that only colored coins wallets understand.

akFqRqfdmAaXfPDmvQZVpcAQnQZmqrx4gcZ

Now, you can take a look on an Open Asset compatible wallet like Coinprism, and see my asset correctly detected:

As I have told you before, the Asset ID is derived from the issuer’s **ScriptPubKey**, here is how to get it in code:

AVAVfLSb1KZf9tJzrUVpktjxKUXGxUTD4e