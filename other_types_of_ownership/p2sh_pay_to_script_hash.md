## P2SH \(Pay To Script Hash\) {#p2sh-pay-to-script-hash}

Pengkodean Multi-Sig cukup mudah, dan dapat bekerja. Namun sebelum p2sh, belum ada cara yang bisa dilakukan untuk meminta kepada customer agar membayarnya menggunakan`scriptPubKey` multi-sig dengan mudah, semudah menggunakan`BitcoinAddress`.

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

Karena "payer" hanya mengetahui tentang **Hash dari RedeemScript**, dia tidak mengetahui **Redeem Script**, oleh karena itu, dalam hal ini, kita juga tidak mengetahui apakah ia mengirimkan uangnya menggunakan multi sig dari transaksi antara Bob\/Satoshi\/Alice.

Dalam hal penandatanganan transaksi, mirip dengan yang telah kita lakukan sebelumnya. Hanya saja perbedaannya adalah, anda terlebih dahulu harus menentukan **Redeem Script** ketika anda membangun koin itu pada **TransactionBuilder**

Mari kita bayangkan kembali bahwa multi sig P2SH menerima koin dalam sebuah transaksi, atau yang disebut dengan`received`.

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

> Warning: Pembayaran ini dikirimkan kepada `redeemScript.Hash` bukan kepada `redeemScript`!

Lalu, jika alice\/bob\/satoshi ingin dapat membelanjakan koin yang telah diterimanya, mereka bisa membuat`ScriptCoin`.

```cs
//Give the redeemScript to the coin for Transaction construction
//and signing
ScriptCoin coin = received.Outputs.AsCoins().First()
                                    .ToScriptCoin(redeemScript);
```

![](../assets/ScriptCoin.png)

Sisa kode selanjutnya untuk generate transaksi dan juga penandatangan transaksi, sama persis pada pembahasan sebelumnya tentang multi sig.

