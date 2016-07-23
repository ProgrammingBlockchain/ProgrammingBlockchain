## Proof of ownership Sebagai Metode Autentifikasi {#proof-of-ownership-as-an-authentication-method}

> \[[2016.05.02](https://www.youtube.com/watch?v=dZNtbAFnr-0)\] Nama saya adalah Craig Wright dan saya akan mendemonstrasikan penandatanganan sebuah pesan dengan public key yang merupakan transaksi pertama kali yang pernah dilakukan di dalam Bitcoin.

```cs
var bitcoinPrivateKey = new BitcoinSecret("XXXXXXXXXXXXXXXXXXXXXXXXXX");

var message = "I am Craig Wright";
string signature = bitcoinPrivateKey.PrivateKey.SignMessage(message);
Console.WriteLine(signature); // IN5v9+3HGW1q71OqQ1boSZTm0/DCiMpI8E4JB1nD67TCbIVMRk/e3KrTT9GvOuu3NGN0w8R2lWOV2cxnBp+Of8c=
```

Apa itu cukup sulit?

Anda mungkin masih ingat Craig Wright, yang berusaha membuat kita percaya bahwa dialah Satoshi Nakamoto.  
Dia telah berhasil meyakinkan segelintir orang Bitcoin yang berpengaruh, para wartawan dengan melakukan beberapa _social engineering_.   
Untungnya tanda tangan digital \(digital signatures\) tidak bisa bekerja dengan cara itu.  
Mari kita lihat address bitcoin pertama di Internet, yang terkait dengan [genesis block](https://en.bitcoin.it/wiki/Genesis_block): [1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa](https://blockchain.info/address/1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa) lalu verifikasi klaim:

```cs
var message = "I am Craig Wright";
var signature = "IN5v9+3HGW1q71OqQ1boSZTm0/DCiMpI8E4JB1nD67TCbIVMRk/e3KrTT9GvOuu3NGN0w8R2lWOV2cxnBp+Of8c=";

var address = new BitcoinPubKeyAddress("1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa");
bool isCraigWrightSatoshi = address.VerifyMessage(message, signature);

Console.WriteLine("Is Craig Wright Satoshi? " + isCraigWrightSatoshi);
```

SPOILER ALERT! The bool will be false.

Berikut bagaimana anda membuktikan bahwa anda adalah pemilik address tanpa harus memindah koin:

**Address:**  
[1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB](https://blockchain.info/address/1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB)  
**Message:**  
Nicolas Dorier Book Funding Address  
**Signature:**  
H1jiXPzun3rXi0N9v9R5fAWrfEae9WPmlL5DJBj1eTStSvpKdRR8Io6\/uT9tGH\/3OnzG6ym5yytuWoA9ahkC3dQ=

Ini adalah bukti bahwa Nicolas Dorier memiliki private key dari address itu.   
**Latihan:** Coba pastikan bahwa Nicolas tidak berbohong!

### Sidenote

Mungkin anda tahu bagaimana cara kerja PGP? Cukup mirip bukan?   
Mungkin ini bisa menjadi dasar alternatif PGP yang _user friendly._  
Silahkan membangunnya di NBitcoin:-\)

