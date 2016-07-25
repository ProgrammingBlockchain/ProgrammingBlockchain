## P2SH \(Pay To Script Hash\) {#p2sh-pay-to-script-hash}

Pengkodean Multi-Sig cukup mudah untuk membuatnya bisa bekerja. Namun sebelum p2sh, belum ada cara yang bisa dilakukan untuk meminta kepada customer agar membayarnya menggunakan`scriptPubKey` multi-sig dengan mudah, semudah menggunakan`BitcoinAddress`.

**P2SH**, atau **Pay To Script Hash**, adalah cara termudah untuk merepresentasikan`scriptPubKey` semudah menggunakan`BitcoinScriptAddress`, meskipun untuk mewujudkannya juga cukup kompleks dan rumit.

Pada pembahasan sebelumnya, kita menggunakan berikut ini untuk generate multisig:

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

Cukup rumit bukan?

Sebaliknya, mari kita lihat bagaimana pada`scriptPubKey` akan terlihat sebagai pembayaran dengan **P2SH**.

```cs
Key bob = new Key();
Key alice = new Key();
Key satoshi = new Key();

var paymentScript = PayToMultiSigTemplate
    .Instance
    .GenerateScriptPubKey(2, new[] { bob.PubKey, alice.PubKey, satoshi.PubKey }).PaymentScript;

Console.WriteLine(paymentScript);
```

```
OP_HASH160 57b4162e00341af0ffc5d5fab468d738b3234190 OP_EQUAL
```

Apa sudah bisa membedakan? P2sh`scriptPubKey`merepresentasikan hash dari script multi-sig: `redeemScript.Hash.ScriptPubKey`

Karena itu adalah hash, maka anda juga dapat merubahnya menjadi string base58 dengan mudah, seperti`BitcoinScriptAddress`.

```cs
Key bob = new Key();
Key alice = new Key();
Key satoshi = new Key();

Script redeemScript =
    PayToMultiSigTemplate
    .Instance
    .GenerateScriptPubKey(2, new[] { bob.PubKey, alice.PubKey, satoshi.PubKey });
//Console.WriteLine(redeemScript.Hash.ScriptPubKey);
Console.WriteLine(redeemScript.Hash.GetAddress(Network.Main)); // 3E6RvwLNfkH6PyX3bqoVGKzrx2AqSJFhjo
```

Beberapa address dapat terbaca dengan baik di berbagai klien wallet. Meskipun pada beberapa wallet tidak begitu mengenal “multi sig”.

Pada transaksi pembayaran P2SH, kita merujuk sebagai **Redeem Script**, dimana yang akan di hash adalah`scriptPubKey`.

![](../assets/RedeemScript.png)

Since the payer only knows about the **Hash of the RedeemScript**, he does not know the **Redeem Script**, and so, in our case, don’t even have to know that he is sending money to a multi sig of Bob\/Satoshi\/Alice.

Signing such transaction is similar to what we have done before. The only difference is that you have to provide the **Redeem Script** when you build the Coin for the **TransactionBuilder**

Imagine that the multi sig P2SH receive a coin in a transaction called `received`.

```cs
Script redeemScript =
    PayToMultiSigTemplate
    .Instance
    .GenerateScriptPubKey(2, new[] { bob.PubKey, alice.PubKey, satoshi.PubKey });
////Console.WriteLine(redeemScript.Hash.ScriptPubKey);
//Console.WriteLine(redeemScript.Hash.GetAddress(Network.Main));

Transaction received = new Transaction();
//Pay to the script hash
received.Outputs.Add(new TxOut(Money.Coins(1.0m), redeemScript.Hash));
```

> Warning: The payment is sent to `redeemScript.Hash` and not to `redeemScript`!

Then, once alice\/bob\/satoshi want to spend what they received, instead of creating a `Coin` they create a `ScriptCoin`.

```cs
//Give the redeemScript to the coin for Transaction construction
//and signing
ScriptCoin coin = received.Outputs.AsCoins().First()
                                    .ToScriptCoin(redeemScript);
```

![](../assets/ScriptCoin.png)

The rest of the code concerning transaction generation and signing is exactly the same as in the previous part with native multi sig.

