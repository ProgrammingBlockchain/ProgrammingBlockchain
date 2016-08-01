## P2PK\[H\] \(Pay to Public Key \[Hash\]\) {#p2pk-h-pay-to-public-key-hash}

### P2PKH - Quick recap

Kita telah mempelajari bahwa **Address** **Bitcoin **adalah **hash **dari sebuah **public key**:

```cs
var publicKeyHash = new Key().PubKey.Hash;
var bitcoinAddress = publicKeyHash.GetAddress(Network.Main);
Console.WriteLine(publicKeyHash); // 41e0d7ab8af1ba5452b824116a31357dc931cf28
Console.WriteLine(bitcoinAddress); // 171LGoEKyVzgQstGwnTHVh3TFTgo5PsqiY
```

Kita juga mempelajari beberapa hal mendasar pada blockchain, dan mengetahui juga bahwa tidak ada yang namanya **address bitcoin** sebenarnya. Karena blockchain mengidentifikasi penerima \(receiver\) dengan sebuah **ScriptPubKey**, dan **ScriptPubKey,** yang memungkinkan untuk generate key dari address:

```cs
var scriptPubKey = bitcoinAddress.ScriptPubKey;
Console.WriteLine(scriptPubKey); // OP_DUP OP_HASH160 41e0d7ab8af1ba5452b824116a31357dc931cf28 OP_EQUALVERIFY OP_CHECKSIG
```

dan sebaliknya:

```cs
var sameBitcoinAddress = scriptPubKey.GetDestinationAddress(Network.Main);
```

### P2PK

Namun, semua **ScriptPubKey** tidak merepresentasikan sebuah Address Bitcoin. Sebagai contoh pada transaksi pertama di dalam blockchain, yang disebut dengan genesis:

```cs
Block genesisBlock = Network.Main.GetGenesis();
Transaction firstTransactionEver = genesisBlock.Transactions.First();
var firstOutputEver = firstTransactionEver.Outputs.First();
var firstScriptPubKeyEver = firstOutputEver.ScriptPubKey;
var firstBitcoinAddressEver = firstScriptPubKeyEver.GetDestinationAddress(Network.Main);
Console.WriteLine(firstBitcoinAddressEver == null); // True
```

```cs
Console.WriteLine(firstTransactionEver);
```

```json
{
…
  "out": [
    {
      "value": "50.00000000",
      "scriptPubKey": "04678afdb0fe5548271967f1a67130b7105cd6a828e03909a67962e0ea1f61deb649f6bc3f4cef38c4f35504e51ec112de5c384df7ba0b8d578a4c702b6bf11d5f OP_CHECKSIG"
    }
  ]
}
```

anda dapat melihat bentuk **scriptPubKey** berbeda:

```cs
Console.WriteLine(firstScriptPubKeyEver); // 04678afdb0fe5548271967f1a67130b7105cd6a828e03909a67962e0ea1f61deb649f6bc3f4cef38c4f35504e51ec112de5c384df7ba0b8d578a4c702b6bf11d5f OP_CHECKSIG
```

Address bitcoin address ditunjukkan oleh: **OP\_DUP &lt;hash&gt; OP\_EQUALVERIFY OP\_CHECKSIG** namun disini kita mempunyai : **&lt;pubkey&gt; OP\_CHECKSIG**

Faktanya, di awal, **public key** digunakan secara langsung dalam **ScriptPubKey**.

```cs
var firstPubKeyEver = firstScriptPubKeyEver.GetDestinationPublicKeys().First();
Console.WriteLine(firstPubKeyEver); // 04678afdb0fe5548271967f1a67130b7105cd6a828e03909a67962e0ea1f61deb649f6bc3f4cef38c4f35504e51ec112de5c384df7ba0b8d578a4c702b6bf11d5f
```

Sekarang kita menggunakan hash utama dari public key.

![](../assets/PPKH.png)

```cs
key = new Key();
Console.WriteLine("Pay to public key : " + key.PubKey.ScriptPubKey);
Console.WriteLine();
Console.WriteLine("Pay to public key hash : " + key.PubKey.Hash.ScriptPubKey);
```

```
Pay to public key : 02fb8021bc7dedcc2f89a67e75cee81fedb8e41d6bfa6769362132544dfdf072d4 OP_CHECKSIG
Pay to public key hash : OP_DUP OP_HASH160 0ae54d4cec828b722d8727cb70f4a6b0a88207b2 OP_EQUALVERIFY OP_CHECKSIG
```

2 tipe pembayaran disana merepresentasikan sebagai **P2PK** \(pay to public key\) dan **P2PKH** \(pay to public key hash\).

Satoshi lalu memutuskan untuk menggunakan P2PKH menggantikan P2PK karena dua alasan:

* Elliptic Curve Cryptography, kriptografi yang digunakan untuk **public key** dan **private key**\) anda, rentan untuk dimodifikasi. Shor's algorithm for solving the discrete logarithm problem on elliptic curves. Artinya, dengan komputer quantum, secara teori, memungkinkan pada suatu saat dimasa depan untuk mendapat** sebuah private key dari public key**. Bisa dilakukan hanya dengan mempublish public key, namun itu terjadi ketika ada transaksi pengeluaran saja yang telah dilakukan. Serangan seperti itu menjadi tidak efektif sebenarnya \(dengan asumsi jika address tidak lagi digunakan\). 
* Ukuran hash cukup kecil \(20 byte\), cukup kecil untuk di cetak, dan mudah untuk di embed pada sebuah ruang penyimpanan kecil, seperti pada QR code.

Untuk saat ini, tidak ada alasan untuk menggunakan P2PK secara langsung, namun tetap masih digunakan untuk dikombinasikan dengan P2SH. Tentang hal ini, pernah juga  di diskusikan di forum Reddit:

> \([Discussion](https://www.reddit.com/r/Bitcoin/comments/4isxjr/petition_to_protect_satoshis_coins/d30we6f)\) Jika pada awalnya menggunakan P2PK, tidak difungsikan sebagai address, karena dapat berdampak serius terhadap harga bitcoin.

### Latihan

\([nopara73](https://github.com/nopara73)\) Saat membaca pada pembahasan di bagian ini ada terdapat begitu banyak singkatan, mulai dari P2PK, P2PKH, P2W, dan lain sebagainya..\) cukup membingungkan.  
Trik yang saya lakukan adalah dengan mengucapkan istilah singkatan-singkatan tersebut sepenuhnya _\(membaca penuh singkatan tersebut, misal P2PK, dibaca Pay to Public Key\)_ di setiap pembahasan berikutnya. Tiba-tiba semuanya menjadi lebih masuk akal dan bisa dipahami. Saya menyarankan anda untuk melakukan hal yang sama. 

