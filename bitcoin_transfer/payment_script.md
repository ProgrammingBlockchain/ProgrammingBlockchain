## ScriptPubKey {#payment-script}

Anda mungkin tidak mengetahui, bahwa sebenarnya tidak ada yang disebut dengan Address Bitcoin. Karena sebenarnya di dalam protokol Bitcoin, hanya mengidentifikasi penerima Bitcoin dengan menggunakan **ScriptPubKey **saja.

![](../assets/ScriptPubKey.png)  
**ScriptPubKey** nampak seperti di bawah ini:  
`OP_HASH160 14836dbe7f38c5ac3d49e8d790af808a4ee9edcf OP_EQUALVERIFY OP_CHECKSIG`

Ini adalah script singkat yang menjelaskan mengapa ada beberapa persyaratan yang harus terpenuhi untuk bisa mengklaim kepemilikan bitcoin. Kita akan mencoba bagaimana penerapan **ScriptPubKey** ini dapat bekerja.

ScriptPubKey bisa digenerate dari Address Bitcoin. Langkah ini dilakukan juga pada hampir semua klien Bitcoin. Agar menjadi lebih mudah _\(human friendly\)_. Sehingga Address Bitcoin ke Blockchain dapat dibaca.

![](../assets/BitcoinAddressToScriptPubKey.png)

```cs
var publicKeyHash = new KeyId("14836dbe7f38c5ac3d49e8d790af808a4ee9edcf");

var testNetAddress = publicKeyHash.GetAddress(Network.TestNet);
var mainNetAddress = publicKeyHash.GetAddress(Network.Main);

Console.WriteLine(mainNetAddress.ScriptPubKey); // OP_DUP OP_HASH160 14836dbe7f38c5ac3d49e8d790af808a4ee9edcf OP_EQUALVERIFY OP_CHECKSIG
Console.WriteLine(testNetAddress.ScriptPubKey); // OP_DUP OP_HASH160 14836dbe7f38c5ac3d49e8d790af808a4ee9edcf OP_EQUALVERIFY OP_CHECKSIG
```

Perhatikan di **ScriptPubKey** untuk testnet dan mainnet apakah addressnya sama?  
Perhatikan di **ScriptPubKey** terdapat hash public key?  
Kita masih belum melangkah ke rincian detailnya, namun perhatikan disana bahwa **ScriptPubKey** tidak ada hubungannya dengan Address Bitcoin, melainkan hanya menunjukkan hash public key saja.

Address Bitcoin terdiri dari versi byte yang mengidentifikasi jaringan mana yang digunakan address dan hash public key tersebut. Jadi kita bisa melihat lagi kebelakang, generate address bitcoin itu dari **ScriptPubKey** dan _network identifier_.

```cs
var paymentScript = publicKeyHash.ScriptPubKey;
var sameMainNetAddress = paymentScript.GetDestinationAddress(Network.Main);
Console.WriteLine(mainNetAddress == sameMainNetAddress); // True
```

Hal itu memungkinkan juga untuk mengambil hal dari ScriptPubKey, dan generate Address Bitcoin:

```cs
var samePublicKeyHash = (KeyId) paymentScript.GetDestination();
Console.WriteLine(publicKeyHash == samePublicKeyHash); // True
var sameMainNetAddress2 = new BitcoinPubKeyAddress(samePublicKeyHash, Network.Main);
Console.WriteLine(mainNetAddress == sameMainNetAddress2); // True
```

> **Catatan:** ScriptPubKey tidak harus selalu berisi hash public key\(s\), yang mengijinkan transaksi pengeluaran.

Jadi sekarang anda bisa mengetahui relasi antara Private Key, Public Key, Public Key Hash, Bitcoin Address dan ScriptPubKey.

In the reminder of this book, we will exclusively use **ScriptPubKey**. A Bitcoin Address is only a user interface concept.

