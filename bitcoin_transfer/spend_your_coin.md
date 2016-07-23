## Membelanjakan koin anda {#spend-your-coin}

Setelah anda mengetahui apa itu **address** **bitcoin**, **ScriptPubKey**, **private key**, dan **penambang \(miner\), **maka anda juga bisa membuat transaksi pertama anda.

Mari kita mulai dengan melihat sebuah **transaksi** yang mempunyai **TxOut, **yang ingin anda transaksikan. seperti yang telah kita pelajari sebelumnya:

Buat sebuah **project baru** \(&gt;.net45\) lalu install **QBitNinja.Client** NuGet.

Apakah anda sebelumnya telah generate private key sendiri? Apakah anda telah mengirim sejumlah bitcoin pada address tersebut? jika belum, jangan khawatir, kita akan mengulangi cara tersebut, sehingga anda juga bisa melakukannya:

```cs
var network = Network.Main;

var privateKey = new Key();
var bitcoinPrivateKey = privateKey.GetWif(network);
var address = bitcoinPrivateKey.GetAddress();

Console.WriteLine(bitcoinPrivateKey);
Console.WriteLine(address);
```

Catat **PrivateKey **bitcoin, **address**, kirimkan sejumlah koin disana, dan catat juga ID transaksinya. Anda bisa cek ID transaksi itu di wallet anda, atau di blockexplorer seperti di blockchain.info.

Import private key:

```cs
var bitcoinPrivateKey = new 
BitcoinSecret("cSZjE4aJNPpBtU6xvJ6J4iBzDgTmzTjbq8w2kqnYvAprBCyTsG4x");
var network = bitcoinPrivateKey.Network;
var address = bitcoinPrivateKey.GetAddress();

Console.WriteLine(bitcoinPrivateKey); // cSZjE4aJNPpBtU6xvJ6J4iBzDgTmzTjbq8w2kqnYvAprBCyTsG4x
Console.WriteLine(address); // mzK6Jy5mer3ABBxfHdcxXEChsn3mkv8qJv
```

Lalu ambil informasi transaction:

```cs
var client = new QBitNinjaClient(network);
var transactionId = uint256.Parse("e44587cf08b4f03b0e8b4ae7562217796ec47b8c91666681d71329b764add2e3");
var transactionResponse = client.GetTransaction(transactionId).Result;

Console.WriteLine(transactionResponse.TransactionId); // e44587cf08b4f03b0e8b4ae7562217796ec47b8c91666681d71329b764add2e3
Console.WriteLine(transactionResponse.Block.Confirmations);
```

Sekarang kita telah mempunyai semua informasi yang dibutuhkan untuk membuat transaksi. Ada sejumlah pertanyaan umum tentang transaksi yang berlangsung: **darimana, kemana, dan berapa jumlahnya?**

### Dari mana?

Dalam hal ini, kita coba mengambil opsi yang kedua pada outpoint. Caranya seperti ini:

```cs
var receivedCoins = transactionResponse.ReceivedCoins;
OutPoint outPointToSpend = null;
foreach (var coin in receivedCoins)
{
    if (coin.TxOut.ScriptPubKey == bitcoinPrivateKey.ScriptPubKey)
    {
        outPointToSpend = coin.Outpoint;
    }
}
if(outPointToSpend == null)
    throw new Exception("TxOut doesn't contain our ScriptPubKey");
Console.WriteLine("We want to spend {0}. outpoint:", outPointToSpend.N + 1);
```

Untuk transaksi pembayaran, anda perlu mereferensikan outpoin di dalam transaksi. Anda bisa membuat transaksi itu seperti ini:

```cs
var transaction = new Transaction();
transaction.Inputs.Add(new TxIn()
{
    PrevOut = outPointToSpend
});
```

### Kemana?

Untuk menjawab beberapa pertanyaan **darimana, kemana, dan berapa jumlahnya?**   
Buatlah **TxIn** dan tambahkan ke dalam transaksi. Dan itulah jawaban dari pertanyaan "darimana".  
Buatlah **TxOut** dan tambahkan itu kedalam transaksi untuk menjawab dua pertanyaan lainnya.

### Berapa jumlahnya?

Misalnya jika anda ingin mengirim **0.5 BTC** dari sebuah **input transaksi** sedangkan balance anda **1 BTC, **maka sebenarnya anda mengirimkan seluruh koin anda \(**1 BTC**\)!   
Contohnya bisa dilihat pada diagram di bawah, **output transaksi** anda** output** dituliskan **0.5** BTC kepada penerima \(Hall of The Makers\) dan **0.4999** kembali ke anda. Jadi pada dasarnya, saat anda mengirim koin 0,5 BTC, anda mengirimkan semua 1 BTC, baru sisanya dikembalikan. 
Lalu bagaimana dengan yang **0.0001 BTC**? Itu adalah fee miner sebagai insentifnya, karena telah memasukkan transaksi anda ke block baru.

![](../assets/SpendTx.png)

```cs
TxOut hallOfTheMakersTxOut = new TxOut()
{
    Value = new Money((decimal)0.5, MoneyUnit.BTC),
    ScriptPubKey = hallOfTheMakersAddress.ScriptPubKey
};

TxOut changeBackTxOut = new TxOut()
{
    Value = new Money((decimal)0.4999, MoneyUnit.BTC),
    ScriptPubKey = bitcoinPrivateKey.ScriptPubKey
};

transaction.Outputs.Add(hallOfTheMakersTxOut);
transaction.Outputs.Add(changeBackTxOut);
```

Kita juga bisa melakukan beberapa _tuning_ disini.   
Kami telah mencobanya di testnet. Anda bisa melihatnya disini:   [http:\/\/tbtc.blockr.io\/address\/info\/mzK6Jy5mer3ABBxfHdcxXEChsn3mkv8qJv](http://tbtc.blockr.io/address/info/mzK6Jy5mer3ABBxfHdcxXEChsn3mkv8qJv)

```cs
// How much you want to TO
var hallOfTheMakersAmount = new Money(0.5m, MoneyUnit.BTC);
/* At the time of writing the mining fee is 0.05usd
 * Depending on the market price and
 * On the currently advised mining fee,
 * You may consider to increase or decrease it
*/
var minerFee = new Money(0.0001m, MoneyUnit.BTC);
// How much you want to spend FROM
var txInAmount = receivedCoins[(int) outPointToSpend.N].TxOut.Amount;
Money changeBackAmount = txInAmount - hallOfTheMakersAmount - minerFee;
```

Lalu menambahkkan kalkulasi nilai pada TxOuts:

```cs
TxOut hallOfTheMakersTxOut = new TxOut()
{
    Value = hallOfTheMakersAmount,
    ScriptPubKey = hallOfTheMakersAddress.ScriptPubKey
};

TxOut changeBackTxOut = new TxOut()
{
    Value = changeBackAmount,
    ScriptPubKey = bitcoinPrivateKey.ScriptPubKey
};
```

Kemudian menambahkannya ke dalam transaksi:

```cs
transaction.Outputs.Add(hallOfTheMakersTxOut);
transaction.Outputs.Add(changeBackTxOut);
```

### Pesan Di Dalam Blockchain

Sekarang coba tambahkan pesan feedback! Pesan ini tidak boleh lebih dari 40 bytes, atau membuat aplikasi itu crash.  
Pesan feedback ini, dapat dicantumkan di dalam transaksi, dan akan muncul \(setelah transaksi itu dikonfirmasi\).

```cs
var message = "nopara73 loves NBitcoin!";
var bytes = Encoding.UTF8.GetBytes(message);
transaction.Outputs.Add(new TxOut()
{
    Value = Money.Zero,
    ScriptPubKey = TxNullDataTemplate.Instance.GenerateScriptPubKey(bytes)
});
```

Lebih jelasnya, coba lihat contoh keseluruhan transaksi yang telah dilakukan ini, sebelum itu ditandatangani:  
Saya punya 3 **TxOut**, 2 dengan **value**, 1 tanpa **value** \(namun ada pesan\). Anda bisa melihat perbedaan antara **scriptPubKey**s dari sebuah **TxOut**s yang normal, dan **scriptPubKey** dari **TxOut** yang dilampiri pesan:

```json
{
  "hash": "b7803df4b90fd615532bcbdb3b63eb1af5a2e4ae36f29a6fbf9f57d0a1842e0a",
  "ver": 1,
  "vin_sz": 1,
  "vout_sz": 3,
  "lock_time": 0,
  "size": 154,
  "in": [
    {
      "prev_out": {
        "hash": "e44587cf08b4f03b0e8b4ae7562217796ec47b8c91666681d71329b764add2e3",
        "n": 1
      },
      "scriptSig": ""
    }
  ],
  "out": [
    {
      "value": "0.50000000",
      "scriptPubKey": "OP_DUP OP_HASH160 d3a689bc36464b9d74e1721fd321d4686eae594e OP_EQUALVERIFYOP_CHECKSIG"
    },
    {
      "value": "0.62840112",
      "scriptPubKey": "OP_DUP OP_HASH160 ce2c16edb74aef1caa6db0078af9d3a5b8fd12d1 OP_EQUALVERIFYOP_CHECKSIG"
    },
    {
      "value": "0.00000000",
      "scriptPubKey": "OP_RETURN 6e6f706172613733206c6f766573204e426974636f696e21"
    }
  ]
}
```

Lihat lebih detail pada **TxIn**. Kita punya **prev\_out** dan **scriptSig** disana.  
**Latihan:** Coba anda cari bagaimana caranya untuk mendapat **scriptSig** sebelum anda membaca lebih jauh!

Coba lihat **hash** dari **prev\_out** di blockexplorer: [http:\/\/tbtc.blockr.io\/tx\/info\/e44587cf08b4f03b0e8b4ae7562217796ec47b8c91666681d71329b764add2e3](http://tbtc.blockr.io/tx/info/e44587cf08b4f03b0e8b4ae7562217796ec47b8c91666681d71329b764add2e3)  
Di **prev\_out** **n** adalah 1. Sejak kita mengindex dari 0, artinya saya ingin menghabiskannya pada output kedua dari transaksi itu.  
Pada blockexplorer kita bisa dapat melihat address yang sesuai adalah: `mzK6Jy5mer3ABBxfHdcxXEChsn3mkv8qJv` and I can get the scriptSig from the address like this:

```cs
var address = BitcoinAddress.Create("mzK6Jy5mer3ABBxfHdcxXEChsn3mkv8qJv");
transaction.Inputs[0].ScriptSig = address.ScriptPubKey;
```

### Menandatangani Transaksi Anda

Jika anda telah dapat membuat transaksi, maka tentu kita harus bisa menandatanganinya. Artinya, anda harus bisa membuktikan bahwa anda memiliki TxOut yang telah anda referensikan dalam input.

Penandatanganan ini [cukup rumit](https://en.bitcoin.it/w/images/en/7/70/Bitcoin_OpCheckSig_InDetail.png), namun kita coba untuk membuatnya lebih simpel.

Pertama coba kita tengok kembali **scriptSig** dari **in**, bagaimana kita bisa mendapatkan kode itu. Yang perlu diingat, kita  copypasted address tersebut dari blockexplorer. Sekarang, mari kita ambil kode itu dari QBitNinja transactionResponse:

```cs
transaction.Inputs[0].ScriptSig =  bitcoinPrivateKey.ScriptPubKey;
```

Anda harus menggunakan private key anda untuk menandatangani:

```cs
transaction.Sign(bitcoinPrivateKey, false);
```

### Broadcast Transaksi

Selamat, karena anda telah berhasil menandatangani transaksi pertama anda! Sekarang tinggal bagaimana menyebar atau broadcas transaksi itu ke dalam jaringan \(network\), sehingga para penambang bisa mendengar transaksi tersebut.

#### Menggunakan QBitNinja:

```cs
BroadcastResponse broadcastResponse = client.Broadcast(transaction).Result;

if (!broadcastResponse.Success)
{
    Console.WriteLine("ErrorCode: " + broadcastResponse.Error.ErrorCode);
    Console.WriteLine("Error message: " + broadcastResponse.Error.Reason);
}
else
{
    Console.WriteLine("Success! You can check out the hash of the transaciton in any block explorer:");
    Console.WriteLine(transaction.GetHash());
}
```

#### Menggunakan Bitcoin Core Anda:

```cs
using (var node = Node.ConnectToLocal(network)) //Connect to the node
{
    node.VersionHandshake(); //Say hello
                             //Advertize your transaction (send just the hash)
    node.SendMessage(new InvPayload(InventoryType.MSG_TX, transaction.GetHash()));
    //Send it
    node.SendMessage(new TxPayload(transaction));
    Thread.Sleep(500); //Wait a bit
}
```

Menggunakan kode block itu akan menangani koneksi yang terputus ke node. Itu saja.

Anda juga dapat terhubung ke jaringan bitcoin secara langsung. Namun lebih baik anda terhubung pada node anda sendiri, agar lebih cepat dan mudah. 

