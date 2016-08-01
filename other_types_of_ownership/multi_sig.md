## Multi Sig {#multi-sig}

Multi Sig ini memungkinkan untuk sharing kepemilikan \(ownership\) koin.  
Untuk itu, anda perlu membuat sebuah`ScriptPubKey` yang merepresentasikan sebuah **m-of-n multi sig.** Maksudnya, untuk mentransaksikan koin, **m** private key dibutuhkan untuk menandatangani pada publik key **n**.

Mari kita coba membuat sebuah multi sig pada contoh transaksi antara Bob, Alice, dan Satoshi, dimana dua dari ketiga orang tersebut membutuhkannya untuk mentransaksikan koin.

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

Seperti yang dapat anda lihat,`scriptPubkey` mempunyai bentuk: `<sigsRequired> <pubkeys…> <pubKeysCount> OP_CHECKMULTISIG`

Proses penandatanganan tersebut sedikit lebih rumit daripada hanya`Transaction.Sign`, yang mungkin tidak dapat bekerja pada multi sig.

Meski kita membicarakan lebih dalam tentang hal ini, lebih baiknya kita menggunakan`TransactionBuilder` untuk menandatangani transaksinya.

Bayangkan saja pada multi-sig `scriptPubKey` menerima koin transaksi, yang disebut dengan`received`:

```cs
var received = new Transaction();
received.Outputs.Add(new TxOut(Money.Coins(1.0m), scriptPubKey));
```

Bob dan Alice setuju untuk membayar Nico 1.0 BTC untuk jasanya.
Jadi untuk mendapat`Coin` mereka menerimanya dalam sebuah transaksi:

```cs
Coin coin = received.Outputs.AsCoins().First();
```

![](../assets/coin.png)

Lalu, dengan `TransactionBuilder`, membuat sebuah **unsigned transaction**.

```cs
BitcoinAddress nico = new Key().PubKey.GetAddress(Network.Main);
TransactionBuilder builder = new TransactionBuilder();
Transaction unsigned = 
    builder
      .AddCoins(coin)
      .Send(nico, Money.Coins(1.0m))
      .BuildTransaction(sign: false);
```

Karena transaksi itu masih belum ditandatangani, maka begini cara Alice menandatanganinya:

```cs
Transaction aliceSigned =
    builder
        .AddCoins(coin)
        .AddKeys(alice)
        .SignTransaction(unsigned);
```

![](../assets/aliceSigned.png)

Dilanjutkan dengan Bob:

```cs
Transaction bobSigned =
    builder
        .AddCoins(coin)
        .AddKeys(bob)
        .SignTransaction(aliceSigned);
```

![](../assets/bobSigned.png)

Sekarang, Bob dan Alice dapat menggabungkan tanda tangan mereka pada satu transaksi saja.

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

Transaksi itupun sekarang siap untuk dimasukkan ke dalam jaringan.

Meski jaringan Bitcoin support untuk multi sig seperti yang dijelaskan di sini, ada satu pertanyaan yang penting: Bagaimana kita bertanya kepada pengguna yang tidak mengetahui cara untuk melakukan pembayaran dengan multi sig seperti pada transaksi antara satoshi\/alice\/bob, karena`scriptPubKey`tidak bisa merepresentasikan address bitcoin dengan mudah?

Tidakkah anda berfikir akan menjadi bagus jika hal tersebut bisa merepresentasikan`scriptPubKey` dengan mudah sebagai Address Bitcoin?

Hal itu memungkinkan bisa dilakukan, disebut dengan **Bitcoin Script Address** atau disebut juga Pay to Script Hash. \(P2SH\)

Saat ini, **native Pay To Multi Sig** seperti yang anda lihat, dan **native P2PK**, belum pernah digunakan secara langsung, karena telah dibungkus menjadi **Pay To Script Hash**.

