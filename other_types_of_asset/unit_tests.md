## Unit Test {#unit-tests}

Pembahasan sebelumnya, saya telah hardcoded properti **ColoredCoin**.  
Alasannya adalah karena saya ingin menunjukkan bagaimana membangun **transaksi** keluar dari **ColoredCoin**.

Dalam kenyataannya, anda mungkin lebih baik bergantung pada API pihak ketiga untuk dapat fetch transaksi colored coins maupun untuk sekedar cek balance. Hal tersebut mungkiin bukanlah ide yang bagus, karena artinya anda bergantung dan mempercayakannya pada pihak ketiga.

**NBitcoin** memungkinkan anda untuk tidak bergantung pada layanan web, karena anda dapat mengimplementasikannya sendiri. Anda dapat menggunakan cara yang lebih fleksibel untuk menguji unit test kode anda, dengan menggunakan implementasi anda sendiri.

Mari kita memperkenalkan dua emiten penerbit: Silver dan Gold. Beserta tiga partisipan: Bob, Alice dan Satoshi.  
Jadi kita memulai dengan membuat sebuah transaksi  palsu untuk dapat memberikan beebrapa bitcoin ke Silver, Gold dan Satoshi.

```cs
var gold = new Key();
var silver = new Key();
var goldId = gold.PubKey.ScriptPubKey.Hash.ToAssetId();
var silverId = silver.PubKey.ScriptPubKey.Hash.ToAssetId();

var bob = new Key();
var alice = new Key();
var satoshi = new Key();

var init = new Transaction()
{
    Outputs =
    {
        new TxOut("1.0", gold),
        new TxOut("1.0", silver),
        new TxOut("1.0", satoshi)
    }
};
```

**Init** tidak mengandung penerbitan Colored Coin issuance dan juga Transfer. Tapi bayangkan jika anda ingin dapat memastikan hal tersebut?

Dalam **NBitcoin**, ringkasan transfer dan penerbitan itu digambarkan oleh sebuah class disebut dengan **ColoredTransaction**.

![](../assets/ColoredTransaction.png)

Anda dapat melihat class **ColoredTransaction**:

* **TxIn** untuk pengeluaran
* **TxOut** untuk menerbitkan aset
* **TxOut** untuk transfer 

Namun ada metode yang cukup menarik diperhatikan adalah **FetchColor**, memungkinkan anda untuk dapat mengekstrak informasi transaksi yang anda berikan pada input.

Hal tersebut bergantung pada **IColoredTransactionRepository**.

![](../assets/IColoredCoinTransactionRepository.png)

**IColoredTransactionRepository** hanya menempatkan saja **ColoredTransaction** dari txid. Namun anda dapat melihat hal itu bergantung dari **ITransactionRepository**, yang memetakan id transaksi untuk transaksi itu.

Implementasi dari **IColoredTransactionRepository** adalah **CoinprismColoredTransactionRepository** yang merupakan API publik untuk colored coins.  
Namun anda juga dapat membuat sendiri, berikut bagaimana **FetchColors** bekerja.

Pada kasus yang paling sederhana: **IColoredTransactionRepository** mengetahui warna, dalam beberapa hal, **FetchColors** hanya mengembalikan hasilnya.

![](../assets/FetchColors.png)

Di kasus kedua, **IColoredTransactionRepository** tidak mengetahui apa-apa tentang warna transaksi. 
Jadi **FetchColors** perlu untuk mengkomputasi warna itu sendiri berdasarkan spesifikasi open asset.

Namun, untuk komputasi warna itu, **FetchColors** membutuhkan warna transaksi dari parent.  
Jadi saat fetch di setiap transaksi itu di **ITransactionRepository**, dan memanggil **FetchColors** di setiap transaksinya.   
Setelah **FetchColors** berhasil komputasi warna transaksi parents, lalu menyimpan caches hasil komputasi tersebut ke dalam **IColoredTransactionRepository**.

![](../assets/FetchColors2.png)

Dengan melakukan itu, request fetch warna transaksi dapat diselesaikan dengan cepat.   
Beberapa **IColoredTransactionRepository** adalah read-only \(seperti **CoinprismColoredTransactionRepository, **jadi _Put operation_ disana diabaikan\).

Kembali pada contoh: 
Trik saat menuliskan unit test adalah dengan menggunakan memory **IColoredTransactionRepository**:

```cs
var repo = new NoSqlColoredTransactionRepository();
```

Sekarang, kita dapat mengambil put **init** transaksi.

```cs
repo.Transactions.Put(init);
```

Perhatikan bahwa Put adalah sebuah metode ektensi, jadi anda akan perlu menambahkan ini:

```cs
using NBitcoin.OpenAsset;
```

pada bagian atas file untuk mendapatkan akses.

Jadi sekarang anda dapat mengekstrak warna:

```cs
ColoredTransaction color = ColoredTransaction.FetchColors(init, repo);
Console.WriteLine(color);
```

```json
{
  "inputs": [],
  "issuances": [],
  "transfers": [],
  "destructions": []
}
```

Seperti yang diharapkan, **init** transaksi tidak mempunyai inputs, issuances, transfer atau _destructions_ Colored Coins.

Sekarang, kita gunakan dua koin itu untuk dikirim ke Silver dan Gold sebagai _Issuance Coins_.

```cs
var issuanceCoins =
    init
    .Outputs
    .AsCoins()
    .Take(2)
    .Select((c, i) => new IssuanceCoin(c))
    .OfType<ICoin>()
    .ToArray();
```

Gold pada koin pertama, Silver kedua.

Dari sana, anda dapat mengirim Gold kepada Satoshi dengan **TransactionBuilder**, seperti halnya yang telah kita lakukan di latihan sebelumnya, sedangkan put menempatkan transaksi dalam repositori, dan mencetak hasilnya.

```cs
{
  "inputs": [],
  "issuances": [
    {
      "index": 0,
      "asset": "ATEwaRSNeCgBjxjcur7JtfypFjqQgAtLJs",
      "quantity": 10
    }
  ],
  "transfers": [],
  "destructions": []
}
```

Artinya bahwa pada **TxOut** pertama dikenakan 10 gold.

Sekarang bayangkan jika **Satoshi** ingin dapat mengirim 4 gold kepada **Alice**.  
Pertama, dia akan fetch **ColoredCoin** dari transaksi.

```cs
var goldCoin = ColoredCoin.Find(sendGoldToSatoshi, color).FirstOrDefault();
```

lalu, membangun sebuah transaksi:

```cs
builder = new TransactionBuilder();
var sendToBobAndAlice =
        builder
        .AddKeys(satoshi)
        .AddCoins(goldCoin)
        .SendAsset(alice, new AssetMoney(goldId, 4))
        .SetChange(satoshi)
        .BuildTransaction(true);
```

Kecuali jika anda mendapat pengecualian berupa **NotEnoughFundsException**.  
Alasannya adalah, bahwa transaksi itu terdiri dari 600 satoshi dalam input \(**goldCoin**\), dan 1200 satoshi di  output. \(Satu **TxOut** untuk mengirim aset kepada Alice, dan satunya lagi untuk mengirim kembali sisa pengeluaran kepada Satoshi.\)

Artinya dana anda dianggap tidak cukup mencukupi 600 satoshi tersebut.  
Anda dapat memperbaiki masalah itu dengan menambahkan koin terakir dari 1 BTC dalam **init** transaksi yang dimiliki **satoshi**.

```cs
var satoshiBtc = init.Outputs.AsCoins().Last();
builder = new TransactionBuilder();
var sendToAlice =
        builder
        .AddKeys(satoshi)
        .AddCoins(goldCoin, satoshiBtc)
        .SendAsset(alice, new AssetMoney(goldId, 4))
        .SetChange(satoshi)
        .BuildTransaction(true);
repo.Transactions.Put(sendToAlice);
color = ColoredTransaction.FetchColors(sendToAlice, repo);
```

Mari kita lihat transaksi tersebut pada bagian warna:

```cs
Console.WriteLine(sendToAlice);
Console.WriteLine(color);
```

```json
{
  ….
  "in": [
    {
      "prev_out": {
        "hash": "46117f3ef44f2dfd87e0bc3f461f48fe9e2a3a2281c9b3802e339c5895fc325e",
        "n": 0
      },
      "scriptSig": "304502210083424305549d4bb1632e2c67736383558f3e1d7fb30ce7b5a3d7b87a53cdb3940220687ea53db678b467b98a83679dec43d27e89234ce802daf14ed059e7a09557e801 03e232cda91e719075a95ede4c36ea1419efbc145afd8896f36310b76b8020d4b1"
    },
    {
      "prev_out": {
        "hash": "aefa62270999baa0d57ddc7d2e1524dd3828e81a679adda810657581d7d6d0f6",
        "n": 2
      },
      "scriptSig": "30440220364a30eb4c8a82cc2a79c54d0518b8ba0cf4e49c73a5bbd17fe1a5683a0dfa640220285e98f3d336f1fa26fb318be545162d6a36ce1103c8f6c547320037cb1fb8e901 03e232cda91e719075a95ede4c36ea1419efbc145afd8896f36310b76b8020d4b1"
    }
  ],
  "out": [
    {
      "value": "0.00000000",
      "scriptPubKey": "OP_RETURN 4f41010002060400"
    },
    {
      "value": "0.00000600",
      "scriptPubKey": "OP_DUP OP_HASH160 5bb41cd29f4e838b4b0fdcd0b95447dcf32c489d OP_EQUALVERIFY OP_CHECKSIG"
    },
    {
      "value": "0.00000600",
      "scriptPubKey": "OP_DUP OP_HASH160 469c5243cb08c82e78a8020360a07ddb193f2aa8 OP_EQUALVERIFY OP_CHECKSIG"
    },
    {
      "value": "0.99999400",
      "scriptPubKey": "OP_DUP OP_HASH160 5bb41cd29f4e838b4b0fdcd0b95447dcf32c489d OP_EQUALVERIFY OP_CHECKSIG"
    }
  ]
}
Colored :
{
  "inputs": [
    {
      "index": 0,
      "asset": " ATEwaRSNeCgBjxjcur7JtfypFjqQgAtLJs ",
      "quantity": 10
    }
  ],
  "issuances": [],
  "transfers": [
    {
      "index": 1,
      "asset": " ATEwaRSNeCgBjxjcur7JtfypFjqQgAtLJs ",
      "quantity": 6
    },
    {
      "index": 2,
      "asset": " ATEwaRSNeCgBjxjcur7JtfypFjqQgAtLJs ",
      "quantity": 4
    }
  ],
  "destructions": []
}
```

Dan kita telah berhasil membuat unit test yang dapat mentransmisi dan mentransfer aset tanpa adanya pihak ketiga. 

Anda dapat membuat **IColoredTransactionRepository** sendiri, jika anda tidak ingin bergantung layanan dari pihak ketiga. 

Anda dapt menemukan lebih banyak skenario yang lebih komplek di [NBitcoin tests](https://github.com/NicolasDorier/NBitcoin/blob/master/NBitcoin.Tests/transaction_tests.cs), dan juga ada salah satu artikel“[Build them all](http://www.codeproject.com/Articles/835098/NBitcoin-Build-Them-All)”. 

