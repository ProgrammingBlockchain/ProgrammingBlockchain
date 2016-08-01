# HiddenBitcoin: Managemen key \(HD wallet\) {#hiddenbitcoin-keystorage}

\([nopara73](https://github.com/nopara73)\) "Saya mengembangkan privasi yang berorientasi untuk wallet Bitcoin, disebut dengan [HiddenWallet](https://github.com/nopara73/HiddenWallet). Library [HiddenBitcoin](https://github.com/nopara73/HiddenBitcoin) adalah pengenalan dari abstraksi lain antara NBitcoin dan user interface."

Sebuah wallet Bitcoin mempunyai fungsi tiga key utama, dan pada studi kasus ini, akan membahas seputar hal tersebut:

1. Mengamankan penyimpanan key, dan mengelola akses.  
2. Memonitor key, dan key lain pada Blockchain.  
3. Membangun transaksi dan mempublikasikan transaksi tersebut.   

Dalam pembelajaran ini, saya akan berusaha mengatasi fungsi penyimpanan key.   
Jika anda ingin memeriksa kode secara lebih luas dan detail, anda dapat mencari solusi itu di [GitHub](https://github.com/nopara73/HiddenBitcoin).  
Jika anda hanya ingin tahu saja bagaimana cara cepat untuk mengatur itu dan menggunakannya, anda dapat menemukan tutorial _high level_ saya di [CodeProject](http://www.codeproject.com/Articles/1096320/HiddenBitcoin-High-level-Csharp-Bitcoin-wallet-lib).

**High level yang bagaimana sebenarnya?** Menurut pendapat saya, pengembang GUI, desainer tidak harus banyak melakukan kesalahan. Mereka tidak harus tahu tentang input dan output dan juga scriptpubkeys. Mereka harus tetap berkisar pada addresses, privatekeys dan level wallet. NBitcoin akan dapat memudahkan anda.

## Keputusan saat mendesain Key storage

Sekarang adalah waktu yang tepat untuk memberikan anda template yang sesuai dengan keputusan anda untuk membuat sebuah ruang penyimpanan key \(key storage\), dan hal apa yang harus tetap anda ingat saat membangunnya. 

### Hanya menggunakan satu key

Ini hanya sebagai sebuah jalan pintas saja. Tidak terlalu banyak hal yang bisa menjelaskan mengapa cara ini perlu dilakukan. Meski demikian, mungkin saja hal ini akan cocok untuk kebutuhan anda.   
Untuk sebuah contoh saja, ini adalah sebuah ilustrasi buruk dari sebuah wallet Bitcoin yang pernah saya buat, yang hanya menggunakan satu key saja. Saya menyerahkan keputusan kepada anda dengan segala konsekuensinya. 

![](../assets/TransparentWallet2.png)

### Wallet JBOK

Wallet ini adalah singkatan dari **J**ust a **B**unch **O**f **K**eys. Pada saat menuliskan ini, referensi dari klien menggunakan metode ini untuk dapat menyimpan key.   
Masalah dengan wallet ini, pengguna harus mencadangkan \(backup\) wallet secara periodik. Jika anda ingin dapat mengimport key, dropping key, atau juga merubah password, anda mungkin akan membutuhkan beberapa kombinasi dari wallet ini dan wallet deterministik. Saya memutuskan untuk tidak menggunakan wallet ini karena HiddenWallet saya sedang mencoba untuk berinovasi terhadap privasi, dan saya mempunyai struktur wallet yang lebih bagus tanpa menggunakan wallet ini.

### BIP38 \(Bagian 2\) - Generator key dari Untrusted third party 

Untuk mengulangi saja: Ide ini untuk dapat generate PassphraseCode kepada key generator. Sehingga dengan PassphraseCode, dapat digunakan untuk generate key yang terenkripsi atas nama anda, tanpa harus mengetahui password anda, maupun private key anda.   
HiddenWallet adalah sebuah wallet desktop \(mungkin tidak akan berubah untuk sementara waktu\). Oleh karena itu saya tidak perlu menggunakan pihak ketiga untuk dapat melakukan generate key maupun untuk menyimpan key. Saya memutuskan untuk tidak menerapkan dulu hal ini. 

### Wallet SHD

Struktur wallet ini yang telah saya terapkan. Dalam benak saya, wallet itu adalah singkatan dari **S**tealth and **H**ierarchical **D**eterministic wallet. Dan itu manjadi cara yang bagus untuk menjelaskan apa yang coba saya kembangkan.   
Sebelum masuk dan menjelaskan tentang pengkodeannya, saya ingin memberikan catatan terlebih dahulu, bahwa saya hanya mengimplementasikan bagian _**Stealth**_ saja. Karena mudah seperti menggapai sebuah buah di pohon yang rendah. Meski begitu, saya tidak begitu yakin apakah stealth addresses akan banyak digunakan di masa mendatang Bitcoin.

Pada sebuah address stealth, akan nampak seperti dibawah ini:  `waPXAvDCDGv8sXYRY6XDEymDGscYeepXBV5tgSDF1JHn61rzNk4EXTuBfx22J2W9rPAszXFmPXwD2m52psYhXQe5Yu1cG26A7hkPxs`

## Black box

Saya menerapkan sebuah class, dan meyebutnya dengan **Safe**. Menggunakan class tersebut sebagai sebuah intuitif dari black box.

```cs
var network = Network.MainNet;
```

**Network** tersebut diatas bukanlah **NBitcoin.Network**, developer GUI tidak harus tahu apakah itu menggunakan NBitcoin. Anda mempunyai banyak pilihan network yang bisa digunakan di NBitcoin, namun HiddenBitcoin tidak bisa menangani semuanya. Pada saat ini, yang bisa support hanya `MainNet` dan `TestNet`.  
Network \(jaringan\) adalah sebuah _**enum**_, dapat ditemukan dibawah **HiddenBitcoin.DataClasses** namespace.

```cs
string mnemonic;
Safe safe = Safe.Create(out mnemonic, "password", walletFilePath: @"Wallets\hiddenWallet.hid", network);
Console.WriteLine(mnemonic);
```

Anda juga dapat me-load atau recover class safe:

```cs
Safe loadedSafe = Safe.Load("password", walletFilePath: @"Wallets\hiddenWallet.hid");
if (network != loadedSafe.Network)
    throw new Exception("WrongNetwork");

Safe recoveredSafe = Safe.Recover(mnemonic, "password", walletFilePath: @"Wallets\sameHiddenWallet.hid", network);
```

Anda juga bisa mendapat beberapa key dari safe sebagai string:

```cs
Console.WriteLine("Seed private key: " + safe.Seed);
Console.WriteLine("Seed public key: " + safe.SeedPublicKey);
Console.WriteLine("Third child address: " + safe.GetAddress(2));
Console.WriteLine("First child private key: " + safe.GetPrivateKey(0));
Console.WriteLine("Second child private key and the corresponding address: ");
Console.WriteLine(safe.GetPrivateKeyAddressPair(1).PrivateKey);
Console.WriteLine(safe.GetPrivateKeyAddressPair(1).Address);
Console.WriteLine("The stealth address: " + safe.StealthAddress);
Console.WriteLine("Scan and spend private keys for stealth payments:");
Console.WriteLine(loadedSafe.ScanPrivateKey);
Console.WriteLine(loadedSafe.SpendPrivateKey);
```

```
Seed private key: xprv9s21ZrQH143K4RBm26TMm3qwTtR3Eyh22xDEN3TBebgfAvHPPSjxxFnFGDtnNHvqZ7pihGmAc8o9y1UvfEzcxSzyXAnmvTBowCNi69nXsqJ
Seed public key: xpub661MyMwAqRbcGuGE87zN8Bng1vFXeSQsQB8qARroCwDe3icXvz4DW46j7U6fX8NsKhqcxR7K1mDX4gTbtvCGdeJz5M7py3yEqMsjUH2DYhb
Third child address: 17pGpPX1A2sCdqJXsC5BiwdFphFVgJR9nk
First child private key: xprv9ubnoo3dgCYfrWbYBEM71WoBvzwTtQemEdjW836CeWJYunYBskQhq3nrJMvNBCCFpnU5GbgbL1b2QbPHA4rRPESEhqfKzae5oWe7SAMuxAV
Second child private key and the corresponding address:
xprv9ubnoo3dgCYfuE1hVB3F3Sh5YFJUNUjyZ68PDzPNhpmtqWDtD45zucZYMUAjY22HNxaY6tsvGAdJdcyALCMm2mTAvA4pEp1m7y3BSccKY4r
19FHdsj2YT79TuxbWcDMz9opTU28L1memr
The stealth address: vJmuFuLggpgzivm3UUjQguLhMA6C1SnYFJu5N6QkmXYRCU3nG1Ww36VcXy6zXpJvGeVTidxcsu7U19sfB1rxHhzvSNV5eGGLk6G1Cb
Scan and spend private keys for stealth payments:
L5CTS4U27umRfSBu2ztxsyUeMEYzJJk3HvCp3deSQBJWmRSUqCLg
KyXveppF4Xm3KwJgG7EBSi6SxfMTkaDXYYmv7c7xWRcF7yUNpswp
```

**Catatan:** Idealnya seed keys tidak pernah digunakan. Ini adalah praktek yang lebih baik, jika anda mulai beriterasi melalui key, dengan menggunakan _safe_.

## White box

### Safe.Create

```cs
// Creates a mnemonic, a seed, encrypts it and stores in the specified path.
public static Safe Create(out string mnemonic, string password, string walletFilePath, Network network)
{
    var safe = new Safe(password, walletFilePath, network);
    mnemonic = safe.SetSeed(password).ToString();
    safe.Save(password, walletFilePath, network);

    return safe;
}
```

`safe.SetSeed` menciptakan sebuah mnemonic dan mengatur `_seedPrivateKey`. Akhirnya dapat mengembalikan mnemonic, jadi dapat memberikan class tersebut kembali kepada pengguna. 

![](../assets/RootKey.png)

```cs
private ExtKey _seedPrivateKey;
private Mnemonic SetSeed(string password)
{
    var mnemonic = new Mnemonic(Wordlist.English, WordCount.Twelve);

    _seedPrivateKey = mnemonic.DeriveExtKey(password);

    return mnemonic;
}
```

### safe.Save

Saves disini adalah sebuah file wallet, pertanyaannya adalah apa yang kita simpan kedalam file ini?

```json
{
  "EncryptedSeed":"6PYXR8U5Nu9UoGZcU95DWWKCXppKnYBUKyJgze6DX6bQDNwFzNdJApUzXT",
  "ChainCode":"C+2MiZU7R/33bkvgdDqdQp7xx3nXHSIzS6bUgRsnaus=",
  "Network":"MainNet"
}
```

File wallet ini berupa format JSON. 
Kita bisa mendapatkan kode rantai dan private key dari sebuah ExtKey. Ini bisa bekerja dan begitu juga pada cara yang lain. 

```cs
Key privateKey = _seedPrivateKey.PrivateKey;
byte[] chainCode = _seedPrivateKey.ChainCode;
```

Dan pada bagian akhir kita mengenkripsi private key.

![](../assets/EncryptedKey.png)

```cs
string encryptedBitcoinPrivateKeyString = privateKey.GetEncryptedBitcoinSecret(password, _network).ToWif();
string chainCodeString = Convert.ToBase64String(chainCode);
string networkString = network.ToString();
```

### Safe.Load

Mari kita balikkan dengan proses penyimpanan. 

```cs
public static Safe Load(string password, string walletFilePath)
{
    if (!File.Exists(walletFilePath))
        throw new Exception("WalletFileDoesNotExists");

    var walletFileRawContent = WalletFileSerializer.Deserialize(walletFilePath);

    var encryptedBitcoinPrivateKeyString = walletFileRawContent.EncryptedSeed;
    var chainCodeString = walletFileRawContent.ChainCode;

    var chainCode = Convert.FromBase64String(chainCodeString);

    Network network;
    var networkString = walletFileRawContent.Network;
    if (networkString == Network.MainNet.ToString())
        network = Network.MainNet;
    else if (networkString == Network.TestNet.ToString())
        network = Network.TestNet;
    else throw new Exception("NotRecognizedNetworkInWalletFile");

    var safe = new Safe(password, walletFilePath, network);

    var privateKey = Key.Parse(encryptedBitcoinPrivateKeyString, password, safe._network);
    var seedExtKey = new ExtKey(privateKey, chainCode);
    safe._seedPrivateKey = seedExtKey;

    return safe;
}
```

Berikut apa yang terjadi pada constructor Safe:

```cs
private Safe(string password, string walletFilePath, Network network)
{
    SetNetwork(network);

    SetSeed(password, mnemonicString);

    WalletFilePath = walletFilePath;
}
```

### SetNetwork

Di dalam class kita ingin agar bisa bekerja di`NBitcoin.Network`. Jadi mari kita membuat sebuah private member.

```cs
private NBitcoin.Network _network;
private void SetNetwork(Network network)
{
    if (network == Network.MainNet)
        _network = NBitcoin.Network.Main;
    else if (network == Network.TestNet)
        _network = NBitcoin.Network.TestNet;
    else throw new Exception("WrongNetwork");
}
```

### Safe.Recover

```cs
public static Safe Recover(string mnemonic, string password, string walletFilePath, Network network)
{
    var safe = new Safe(password, walletFilePath, network, mnemonic);
    safe.Save(password, walletFilePath, network);
    return safe;
}
```

Agar bisa bekerja, maka kita perlu memperluas constructor:

```cs
private Safe(string password, string walletFilePath, Network network, string mnemonicString = null)
{
    SetNetwork(network);

    if (mnemonicString != null)
    {
        var mnemonic = new Mnemonic(mnemonicString);
        _seedPrivateKey = mnemonic.DeriveExtKey(password);
    }

    WalletFilePath = walletFilePath;
}
```

### Getters

Berikut bagaimana saya bisa memperoleh key. Dalam hal ini, cukup tidak masuk akal jika menggunakan terlalu banyak keypath yang rumit:

```cs
public PrivateKeyAddressPair GetPrivateKeyAddressPair(int index)
{
    var foo = _seedPrivateKey.Derive(index, true).GetWif(_network);
    return new PrivateKeyAddressPair
    {
        PrivateKey = foo.ToWif(),
        Address = foo.ScriptPubKey.GetDestinationAddress(_network).ToWif()
    };
}
```

### Stealth

```cs
private Key _spendPrivateKey => _seedPrivateKey.PrivateKey;
public string SpendPrivateKey => _spendPrivateKey.GetWif(_network).ToWif();
private Key _scanPrivateKey => _seedPrivateKey.Derive(0, hardened: true).PrivateKey;
public string ScanPrivateKey => _scanPrivateKey.GetWif(_network).ToWif();

public string StealthAddress => new BitcoinStealthAddress
    (_scanPrivateKey.PubKey, new[] {_spendPrivateKey.PubKey}, 1, null, _network
    ).ToWif();
```

