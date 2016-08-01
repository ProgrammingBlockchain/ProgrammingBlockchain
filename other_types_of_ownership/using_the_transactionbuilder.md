## Penggunaan TransactionBuilder {#using-the-transactionbuilder}

Anda telah melihat bagaimana **TransactionBuilder** bisa bekerja jika anda telah berhasil menandatangani **P2SH** pertama anda, dan transaksi **multi-sig**.

Kita juga akan melihat bagaimana anda dapat memanfaatkannya secara penuh, untuk menandatangani transaksi yang lebih rumit.

Dengan **TransactionBuilder** anda dapat:

* Membuat berbagai:

  * **P2PK**, **P2PKH**,  
  * **multi-sig**,  
  * **P2WPK**, **P2WSH**.  

* Transaksi pengeluaran **P2SH** di script redem sebelumnya.

* Transaksi pengeluaran seperti di **Stealth Coin** \(DarkWallet\).

* Menyelesaikan permasalahan dan transfer di **Colored Coins** \(open asset, pada bab pembahasan berikutnya\).

* menggabungkan **sebagian penandatangan  transaksi**.  
* Memperkirakan ukuran akhir \(**size\)** dari **unsigned transaction** dan juga biayanya \(**fees\)**.  
* Verifikasi apakah **transaksi** tersebut betul-betul telah ditandatangani \(**fully signed\)**.  

Tujuan **TransactionBuilder** adalah untuk mengambil **Koin** dan **Keys** sebagai input, lalu mengembalikannya menjadi sebuah transaksi yang telah ditandatangani **\(signed\)**, atau sebagian telah ditandatangani \(**partially signed transaction\)**.

![](../assets/SignedTransaction.png)

**TransactionBuilder** akan mencari tahu **Koin **yang akan digunakan untuk dapat ditandatangani dengan sendirinya.

![](../assets/TransactionBuilder.png)

Penggunaan _builder_ ini dilakukan dalam empat langkah:

* Anda mengumpulkan **koin** yang akan dikeluarkan,
* Anda mengumpulkan **Keys** yang anda miliki,
* Anda menghitung berapa jumlah **Uang** yang ingin anda keluarkan untuk **scriptPubKey**,
* Anda membuat dan menandatangani **transaksi**,
* **Tambahan**: Anda dapat memberikan **transaksi** itu untuk orang lain, lalu ia akan menandatanganinya, atau meneruskan pembuatan transaksi tersebut.

Sekarang mari kita mengumpulkan beberapa **koin **tersebut. Untuk itu, mari kita membuat sebuah transaksi palsu dengan mendanai sejumlah dana ke dalamnya.   
Katakanlah pada **transaksi** tersebut mempunyai sebuah **P2PKH**, **P2PK**, dan koin **multi-sig** dari Bob dan Alice.

```cs
// Create a fake transaction
var bob = new Key();
var alice = new Key();

Script bobAlice = 
    PayToMultiSigTemplate.Instance.GenerateScriptPubKey(
        2, 
        bob.PubKey, alice.PubKey);

var init = new Transaction();
init.Outputs.Add(new TxOut(Money.Coins(1m), bob.PubKey)); // P2PK
init.Outputs.Add(new TxOut(Money.Coins(1m), alice.PubKey.Hash)); // P2PKH
init.Outputs.Add(new TxOut(Money.Coins(1m), bobAlice));
```

Sekarang mari kita katakan bahwa mereka ingin menggunakan`coins` pada transaksi itu untuk membayar Satoshi.

```cs
var satoshi = new Key();
```

Pertama, mereka harus mengumpulkan **Koin**.

```cs
Coin[] coins = init.Outputs.AsCoins().ToArray();
Coin bobCoin = coins[0];
Coin aliceCoin = coins[1];
Coin bobAliceCoin = coins[2];
```

Jadi sekarang katakanlah semisal `bob` ingin mengirim 0.2 BTC, `alice` 0.3 BTC, dan mereka berdua setuju untuk menggunakan`bobAlice` untuk mengirim total 0.5 BTC.

```cs
var builder = new TransactionBuilder();
Transaction tx = builder
        .AddCoins(bobCoin)
        .AddKeys(bob)
        .Send(satoshi, Money.Coins(0.2m))
        .SetChange(bob)
        .Then()
        .AddCoins(aliceCoin)
        .AddKeys(alice)
        .Send(satoshi, Money.Coins(0.3m))
        .SetChange(alice)
        .Then()
        .AddCoins(bobAliceCoin)
        .AddKeys(bob, alice)
        .Send(satoshi, Money.Coins(0.5m))
        .SetChange(bobAlice)
        .SendFees(Money.Coins(0.0001m))
        .BuildTransaction(sign: true);
```

Kemudian anda dapat memverifikasi tanda tangan itu sepenuhnya, dan siap mengirim transaksi itu ke dalam jaringan.

```cs
Console.WriteLine(builder.Verify(tx)); // True
```

Hal yang menyenangkan dari model ini adalah, karena dapat bekerja dengan cara yang sama untuk **P2SH, P2WSH, P2SH\(P2WSH\)**, dan **P2SH\(P2PKH\), **kecuali jika anda perlu untuk membuat **ScriptCoin**.

![](../assets/ScriptCoinFromCoin.png)

```cs
init = new Transaction();
init.Outputs.Add(new TxOut(Money.Coins(1.0m), bobAlice.Hash));

coins = init.Outputs.AsCoins().ToArray();
ScriptCoin bobAliceScriptCoin = coins[0].ToScriptCoin(bobAlice);
```

Lalu signature:

```cs
builder = new TransactionBuilder();
tx = builder
        .AddCoins(bobAliceScriptCoin)
        .AddKeys(bob, alice)
        .Send(satoshi, Money.Coins(0.9m))
        .SetChange(bobAlice.Hash)
        .SendFees(Money.Coins(0.0001m))
        .BuildTransaction(true);
Console.WriteLine(builder.Verify(tx)); // True
```

Untuk **Stealth Coin, **pada dasarnya sama. Kecuali memang, jika anda masih ingat pada bab perkenalan tentang Dark Wallet, saya mengatakan bahwa anda membutuhkan sebuah **ScanKey** untuk melihat **StealthCoin**.

![](../assets/StealthCoin.png)

Kemudian mari dilanjutkan dengan membuat address _stealth darkAliceBob_ seperti di pembahasan sebelumnya:

```cs
Key scanKey = new Key();
BitcoinStealthAddress darkAliceBob =
    new BitcoinStealthAddress
        (
            scanKey: scanKey.PubKey,
            pubKeys: new[] { alice.PubKey, bob.PubKey },
            signatureCount: 2,
            bitfield: null,
            network: Network.Main
        );
```

Jika seseorang mengirim transaksi ini:

```cs
//Someone sent to darkAliceBob
init = new Transaction();
darkAliceBob
    .SendTo(init, Money.Coins(1.0m));
```

Scanner akan mendeteksi StealthCoin:

```cs
//Get the stealth coin with the scanKey
StealthCoin stealthCoin
    = StealthCoin.Find(init, darkAliceBob, scanKey);
```

Dan meneruskan kepada bob dan alice, yang akan menandatangani:

```cs
//Spend it
tx = builder
        .AddCoins(stealthCoin)
        .AddKeys(bob, alice, scanKey)
        .Send(satoshi, Money.Coins(0.9m))
        .SetChange(bobAlice.Hash)
        .SendFees(Money.Coins(0.0001m))
        .BuildTransaction(true);
Console.WriteLine(builder.Verify(tx)); // True
```

> **Catatan:** Anda membutuhkan scanKey untuk transaksi pengeluaran StealthCoin

