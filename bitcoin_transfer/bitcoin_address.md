## Address Bitcoin {#bitcoin-address}

Anda tentu sudah mengetahui bahwa dengan **Address** **Bitcoin**, digunakan untuk bisa menerima pembayaran dari orang lain. Dan anda bisa memberitahukan address bitcoin anda ini kepada orang lain.   
![](../assets/BitcoinAddress.png)  
Dan anda tentu sudah mengetahui, bahwa **private key** digunakan untuk dapat membelanjakan bitcoin yang anda miliki.   
![](../assets/PrivateKey.png)

kedua _key_ ini tidak disimpan ke dalam network, keduanya juga bisa di generated tanpa harus terhubung dengan Internet.

Cara generate key itu adalah dengan menggunakan kode ini:

```cs
Key privateKey = new Key(); // generate a random private key
```

Dari generate private key, kita menggunakan fungsi kriptografi searah _\(one-way\)_, untuk selanjutnya generate sebuah **public key**.

![](../assets/PrivKeyPubKey.png)

```cs
PubKey publicKey = privateKey.PubKey;
Console.WriteLine(publicKey); // 0251036303164f6c458e9f7abecb4e55e5ce9ec2b2f1d06d633c9653a07976560c
```

**Networks **Bitcoin ada dua:

* **TestNet** adalah network Bitcoin untuk pengembangan saja. Bitcoins di network ini tidak bernilai apa-apa.  
* **MainNet** adalah network Bitcoin yang digunakan oleh semua orang.  

> **Catatan:** Anda dapat memperoleh koin testnet ini dengan cepat menggunakan **faucets**, cukup typing saja di google dengan keyword "get testnet bitcoins".

Untuk mendapatkan **address bitcoin, **anda bisa mendapatkannya dengan mudah dengan public key anda, pada **network** yang anda gunakan untuk address tersebut. 

![](../assets/PubKeyToAddr.png)

```cs
Console.WriteLine(publicKey.GetAddress(Network.Main)); // 1PUYsjwfNmX64wS368ZR5FMouTtUmvtmTY
Console.WriteLine(publicKey.GetAddress(Network.TestNet)); // n3zWAo2eBnxLr3ueohXnuAa8mTVBhxmPhq
```

**Lebih tepatnya, address** **bitcoin ini terdiri dari versi byte yang berbeda \(di kedua jaringan\) dan byte hash public key anda menggabungkannya, kemudian dikodekan menjadi Base58Check. Lihat dibawah ini:**

![](../assets/PubKeyHashToBitcoinAddress.png)

```cs
var publicKeyHash = publicKey.Hash;
Console.WriteLine(publicKeyHash); // f6889b21b5540353a29ed18c45ea0031280c42cf
var mainNetAddress = publicKeyHash.GetAddress(Network.Main);
var testNetAddress = publicKeyHash.GetAddress(Network.TestNet);
```

> **Fakta:** Hash dari public key ini di generated dengan hashing SHA256 pada the public key, lalu hash juga RIPEMD160 untuk mendapatkan hasilnya, dengan notasi Big Endian. Fungsi tersebut, akan terlihat seperti ini: RIPEMD160\(SHA256\(pubkey\)\)

Pengkodean Base58Check memiliki beberapa fitur yang rapi, seperti checksum untuk mencegah kesalahan ketik dan juga mengurangi karakter-karakter yang ambigu. Seperti halnya '0', dan 'O'. 

Pengkodean Base58Check dari sebuah address untuk memastikan juga, bahwa pengguna wallet bitcoin tidak mengirimkan uang kepada sebuah address yang seharusnya digunakan pada jaringan yang berbeda. 

```cs
Console.WriteLine(mainNetAddress); // 1PUYsjwfNmX64wS368ZR5FMouTtUmvtmTY
Console.WriteLine(testNetAddress); // n3zWAo2eBnxLr3ueohXnuAa8mTVBhxmPhq
```

> **Tips:** Berlatih pemprograman Bitcoin di MainNet membuat kesalahan-kesalahan yang terjadi menjadi lebih berkesan.

