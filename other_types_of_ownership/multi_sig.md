## Multi Sig {#multi-sig}

It is possible to have shared ownership over coins.  
For that you will create a ```ScriptPubKey``` that represents a **m-of-n multi sig**, this means in order to spend the coins, **m** private keys will need to sign on the **n** different public key provided.

Let’s create a multi sig with Bob, Alice, and Satoshi, where two of them are needed to spend a coin.  

```cs
Key bob = new Key();
Key alice = new Key();
Key satoshi = new Key();

var scriptPubKey = PayToMultiSigTemplate
    .Instance
    .GenerateScriptPubKey(2, new[] { bob.PubKey, alice.PubKey, satoshi.PubKey });

Console.WriteLine(scriptPubKey);
```  

```
2 0282213c7172e9dff8a852b436a957c1f55aa1a947f2571585870bfb12c0c15d61 036e9f73ca6929dec6926d8e319506cc4370914cd13d300e83fd9c3dfca3970efb 0324b9185ec3db2f209b620657ce0e9a792472d89911e0ac3fc1e5b5fc2ca7683d 3 OP_CHECKMULTISIG
```  

As you can see, the ```scriptPubkey``` have the following form: ```<sigsRequired> <pubkeys…> <pubKeysCount> OP_CHECKMULTISIG```  

The process for signing it is a little more complicated than just calling ```Transaction.Sign```, which does not work for multi sig.

Even if we will talk more deeply about the subject, let’s use the ```TransactionBuilder``` for signing the transaction.

Imagine the multi-sig ```scriptPubKey``` received a coin in a transaction called ```received```:

```cs
var received = new Transaction();
received.Outputs.Add(new TxOut(Money.Coins(1.0m), scriptPubKey));
```  

Bob and Alice agree to pay Nico 1.0 BTC for his services.
So the get the ```Coin``` they received from the transaction:  

```cs
Coin coin = received.Outputs.AsCoins().First();
```  

![](../assets/coin.png)  

Then, with the ```TransactionBuilder```, create an **unsigned transaction**.  

```cs
BitcoinAddress nico = new Key().PubKey.GetAddress(Network.Main);
TransactionBuilder builder = new TransactionBuilder();
Transaction unsigned = 
    builder
      .AddCoins(coin)
      .Send(nico, Money.Coins(1.0m))
      .BuildTransaction(sign: false);
```  

The transaction is not yet signed. Here is how Alice signs it:  

```cs
Transaction aliceSigned =
    builder
        .AddCoins(coin)
        .AddKeys(alice)
        .SignTransaction(unsigned);
```  

![](../assets/aliceSigned.png)  

And then Bob:  

```cs
Transaction bobSigned =
    builder
        .AddCoins(coin)
        .AddKeys(bob)
        .SignTransaction(aliceSigned);
```  

![](../assets/bobSigned.png)  

Now, Bob and Alice can combine their signature into one transaction.  

```cs
Transaction fullySigned =
    builder
        .AddCoins(coin)
        .CombineSignatures(aliceSigned, bobSigned);
```  

![](../assets/fullySigned.png)  

```cs
Console.WriteLine(fullySigned);
```  

```json
{
  ...
  "in": [
    {
      "prev_out": {
        "hash": "9df1e011984305b78210229a86b6ade9546dc69c4d25a6bee472ee7d62ea3c16",
        "n": 0
      },
      "scriptSig": "0 3045022100a14d47c762fe7c04b4382f736c5de0b038b8de92649987bc59bca83ea307b1a202203e38dcc9b0b7f0556a5138fd316cd28639243f05f5ca1afc254b883482ddb91f01 3044022044c9f6818078887587cac126c3c2047b6e5425758e67df64e8d682dfbe373a2902204ae7fda6ada9b7a11c4e362a0389b1bf90abc1f3488fe21041a4f7f14f1d856201"
    }
  ],
  "out": [
    {
      "value": "1.00000000",
      "scriptPubKey": "OP_DUP OP_HASH160 d4a0f6c5b4bcbf2f5830eabed3daa7304fb794d6 OP_EQUALVERIFY OP_CHECKSIG"
    }
  ]
}

```  
The transaction is now ready to be sent on the network.

Even if the Bitcoin network supports multi sig as explained here, one question worth asking is: How can you ask to a user who has no clue about bitcoin to pay on satoshi/alice/bob multi sig, since such ```scriptPubKey``` can’t be represented by easy to use Bitcoin Address like we have seen before?

Don’t you think it would be cool if we could to represent such ```scriptPubKey``` as easily and compactly as a Bitcoin Address?

Well, this is possible and it is called a **Bitcoin Script Address** also called Pay to Script Hash. (P2SH)

Nowadays, **native Pay To Multi Sig** as you have seen here, and **native P2PK**, are never used directly as such, they are wrapped into **Pay To Script Hash** payment.