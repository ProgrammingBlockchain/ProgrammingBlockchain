## Transaksi {#transaction}

> \([Mastering Bitcoin](https://github.com/bitcoinbook/bitcoinbook/)\) Transaksi adalah bagian terpenting dari sistem bitcoin. Segala sesuatu yang lain di bitcoin dirancang untuk dapat memastikan bahwa transaksi itu dapat dilakukan, disebar ke dalam jaringan, divalidasi, dan kemudian ditambahkan ke ledger \(Blockchain\). Transaksi adalah struktur data yang menyandikan transfer nilai antar pengguna di dalam sistem bitcoin. Setiap transaksi merupakan entri publik di blockchain bitcoin ini. Menjadi sebuah pembukuan transaksi besar secara global.

Sebuah transaksi mungkin tidak punya penerima, atau bahkan mempunyai penerima lebih dari satu. **Begitu juga halnya dengan pengirim**. Pada Blockchain, pengirim dan penerima diabstraksikan dengan ScriptPubKey, seperti yang telah ditunjukkan pada bab sebelumnya.

Jika anda menggunakan Bitcoin Core, transaksi anda akan menampilkan seperti ini:

![](../assets/BitcoinCoreTransaction.png)

Sekarang, kita jadi tertarik dengan munculnya **Transaction ID** pada transaksi diatas. Transaksi ID tersebut diatas adalah:

`f13dc48fb035bbf0a6e989a26b3ecb57b84f85e0836e777d6edf60d87a4a2d94`

> **Catatan:** TransactionId is didefinisikan oleh SHA256\(SHA256\(txbytes\)\)
> 
> **Catatan:** JANGAN menggunakan TransactionId untuk menangani transaksi yang belum dikonfirmasi. TransactionId dapat dimanipulasi sebelum transaksi itu dikonfirmasi. Hal ini disebut dengan “Transaction Malleability.”

Anda dapat melihat transaksi itu pada blockexplorer seperti di blockchain.info: [https:\/\/blockchain.info\/tx\/f13dc48fb035bbf0a6e989a26b3ecb57b84f85e0836e777d6edf60d87a4a2d94](https://blockchain.info/tx/f13dc48fb035bbf0a6e989a26b3ecb57b84f85e0836e777d6edf60d87a4a2d94) 
Namun sebagai seorang developer, anda mungkin menginginkan layanan yang lebih mudah untuk _query_ dan _parse_. 
Sebagai developer C\# dan pengguna NBitcoin, Nicolas Dorier's di [QBit Ninja](http://docs.qbitninja.apiary.io/) akan menjadi pilihan terbaik anda. Ini adalah layanan web API open source, untuk query blockchain dan melacak wallet.   
QBit Ninja tergantung pada [NBitcoin.Indexer](https://github.com/MetacoSA/NBitcoin.Indexer) di relay dari Microsoft Azure Storage. Developer C\# diharapkan menggunakan [NuGet client package.](http://www.nuget.org/packages/QBitninja.Client)

Jika anda melihat di sini: [http:\/\/api.qbit.ninja\/transactions\/f13dc48fb035bbf0a6e989a26b3ecb57b84f85e0836e777d6edf60d87a4a2d94](http://api.qbit.ninja/transactions/f13dc48fb035bbf0a6e989a26b3ecb57b84f85e0836e777d6edf60d87a4a2d94) Anda akan melihat _raw bytes_ dari transaksi anda.

![](../assets/RawTx.png)

Anda dapat parse hex transaksi itu dengan kode berikut:

```cs
Transaction tx = new Transaction("0100000...");
```

Lalu cepatlah menutup tab tersebut, sebelum membuat anda ketakutan. QBit Ninja melakukan queri dari API dan memparse informasi tersebut, jadi anda bisa melanjutkannya dan install **QBitNinja.Client** NuGet package.

![](../assets/QBitNuGet.png)

Query transaksi by id:

```cs
// Create a client
QBitNinjaClient client = new QBitNinjaClient(Network.Main);
// Parse transaction id to NBitcoin.uint256 so the client can eat it
var transactionId = uint256.Parse("f13dc48fb035bbf0a6e989a26b3ecb57b84f85e0836e777d6edf60d87a4a2d94");
// Query the transaction
GetTransactionResponse transactionResponse = client.GetTransaction(transactionId).Result;
```

Tipe **transactionResponse** adalah **GetTransactionResponse**. Dijalankan melalui QBitNinja.Client.Models namespace. Anda bisa mendapatkan jenis **NBitcoin.Transaction** tersebut di sini:

```cs
NBitcoin.Transaction transaction = transactionResponse.Transaction;
```

Sekarang mari kita lihat contoh untuk mendapat kembali ID transaksi itu dengan kedua \_classes \_ini:

```cs
Console.WriteLine(transactionResponse.TransactionId); // f13dc48fb035bbf0a6e989a26b3ecb57b84f85e0836e777d6edf60d87a4a2d94
Console.WriteLine(transaction.GetHash()); // f13dc48fb035bbf0a6e989a26b3ecb57b84f85e0836e777d6edf60d87a4a2d94
```

**GetTransactionResponse** memiliki informasi tambahan tentang transaksi, seperti nilai dan scriptPubKey pada _inputs_ yang akan dibelanjakan dalam sebuah transaksi.

Jadi yang paling beralasi pada hal ini adalah tentang **inputs** dan **outputs**. Anda dapat melihat sejumlah 13.19683492 Bitcoin telah terkirim pada ScriptPubKey:

```cs
List<ICoin> receivedCoins = transactionResponse.ReceivedCoins;
foreach (var coin in receivedCoins)
{
    Money amount = coin.Amount;

    Console.WriteLine(amount.ToDecimal(MoneyUnit.BTC));
    var paymentScript = coin.ScriptPubKey;
    Console.WriteLine(paymentScript);  // It's the ScriptPubKey
    var address = paymentScript.GetDestinationAddress(Network.Main);
    Console.WriteLine(address);
    Console.WriteLine();
}
```

Kami juga menulis beberapa informasi tentang RECEIVED COINS menggunakan QBitNinja's _class_ GetTransactionResponse.
**Latihan**: Tuliskan informasi yang sama tentang SPENT COINS menggunakan _class_ QBitNinja's GetTransactionResponse!

Lalu mari kita lihat bagaimana kita bisa mendapatkan informasi yang sama tentang RECEIVED COINS menggunakan \_class \_NBitcoin's Transaction.

```cs
var outputs = transaction.Outputs;
foreach (TxOut output in outputs)
{
    Money amount = output.Value;

    Console.WriteLine(amount.ToDecimal(MoneyUnit.BTC));
    var paymentScript = output.ScriptPubKey;
    Console.WriteLine(paymentScript);  // It's the ScriptPubKey
    var address = paymentScript.GetDestinationAddress(Network.Main);
    Console.WriteLine(address);
    Console.WriteLine();
}
```

Sekarang mari kita periksa di **inputs**. Jika anda melihatnya, maka disana akan ada referensi output sebelumnya. Jadi pada setiap input dapat menunjukkan pengeluaran sebelumnya yang telah ditransaksikan pada transaksi ini. 

```cs
var inputs = transaction.Inputs;
foreach (TxIn input in inputs)
{
    OutPoint previousOutpoint = input.PrevOut;
    Console.WriteLine(previousOutpoint.Hash); // hash of prev tx
    Console.WriteLine(previousOutpoint.N); // idx of out from prev tx, that has been spent in the current tx
    Console.WriteLine();
}
```

Istilah **TxOut**, **Output** dan **out** maknanya sama.  
Jadi tidak perlu bingung dengan **OutPoint**, akan dijelaskan lebih lanjut.

Singkatnya, TxOut merupakan jumlah bitcoin dan **ScriptPubKey**. \(Penerima\)

![](../assets/TxOut.png)  
Sebagai ilustrasinya, mari kita membuat sebuah txout dengan 21 bitcoin dari ScriptPubKey pertama dalam transaksi kami saat ini:

```cs
Money twentyOneBtc = new Money(21, MoneyUnit.BTC);
var scriptPubKey = transaction.Outputs.First().ScriptPubKey;
TxOut txOut = new TxOut(twentyOneBtc, scriptPubKey);
```

Setiap **TxOut** secara unik menunjuk address di level blockchain oleh ID transaksi yang meliputinya, dan juga index di dalamnya. kami menyebut referensi tersebut dengan **Outpoint**.

![](../assets/OutPoint.png)

Sebagai contoh, **Outpoint** dari **TxOut** dengan 13.19683492 BTC pada transaksi kami adalah \(4788c5ef8ffd0463422bcafdfab240f5bf0be690482ceccde79c51cfce209edd, 0\).

```cs
OutPoint firstOutPoint = spentCoins.First().Outpoint;
Console.WriteLine(firstOutPoint.Hash); // 4788c5ef8ffd0463422bcafdfab240f5bf0be690482ceccde79c51cfce209edd
Console.WriteLine(firstOutPoint.N); // 0
```

Sekarang mari kita lihat lebih dekat pada input \(di **TxIn**\) dari transaksi itu:

![](../assets/TxIn.png)

**TxIn** terdiri dari **Outpoint** pada **TxOut** yang telah dikeluarkan atau dibelanjakan dari **ScriptSig** \( kita dapat melihat disini bahwa ScriptSig berfungsi sebagai “Proof of Ownership”\) Pada transaksi kami yang sebenarnya, ada 9 input.

```cs
Console.WriteLine(transaction.Inputs.Count); // 9
```

Dengan ID transaksi outpoint sebelumnya, kita bisa melihat informasi yang berkaitan dengan transaksi tersebut. 

```cs
OutPoint firstPreviousOutPoint = transaction.Inputs.First().PrevOut;
var firstPreviousTransaction = client.GetTransaction(firstPreviousOutPoint.Hash).Result.Transaction;
Console.WriteLine(firstPreviousTransaction.IsCoinBase); // False
```

Kita bisa untuk terus melanjutkan melacak ID transaksi hingga kembali sampai kepada transaksi coinbase, transaksi koin baru yang ditambang oleh penambang.   
**Latihan:** Ikuti input pertama pada transaksi ini sampai anda mencapai transaksi coinbase!  
Petunjuk: Setelah mencoba mengikuti hingga 30-40 transaksi, saya menyerah.  
Ya, tebakan anda benar, hal itu bukanlah cara yang efektif untuk melakukannya, namun menjadi sebuah latihan yang baik. 

Pada contoh kami, jumlah output adalah 13.19**70**3492 BTC.

```cs
Money spentAmount = Money.Zero;
foreach (var spentCoin in spentCoins)
{
    spentAmount = (Money)spentCoin.Amount.Add(spentAmount);
}
Console.WriteLine(spentAmount.ToDecimal(MoneyUnit.BTC)); // 13.19703492
```

Pada transaksi tersebut, sejumlah 13.19**68**3492 BTC telah diterima.

**Latihan:** Dapatkan jumlah total yang diterima, seperti yang telah saya lakukan dengan sejumlah pengeluaran.

That means 0.0002 BTC \(or 13.19**70**3492 - 13.19**68**3492\) is not accounted for! The difference between the inputs and outputs are called **Transaction Fees** or **Miner’s Fees**. This is the money that the miner collects for including a given transaction in a block.

```cs
var fee = transaction.GetFee(spentCoins.ToArray());
Console.WriteLine(fee);
```

You should note that a **coinbase transaction** is the only transaction whose value of output are superior to the value of input. This effectively correspond to coin creation. So by definition there is no fee in a coinbase transaction. The coinbase transaction is the first transaction of every block.  
The consensus rule enforce that the sum of output's value in the coinbase transaction does not exceed the sum of transaction fees in the block plus the mining reward.

