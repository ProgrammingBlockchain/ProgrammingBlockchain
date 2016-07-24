# Generate Key dan Enkripsi {#key-generation-encryption}

## Sudah cukup acak? {#is-it-random-enough}

Ketika anda membuat **new Key\(\)**, anda bisa menggunakan PRNG \(Pseudo-Random-Number-Generator\) untuk generate private key. Di windows, menggunakan **RNGCryptoServiceProvider**, sebuah wrapper .NET  API Crypto Windows.

Jika di Android, saya gunakan **SecureRandom**, dan nyatanya, anda juga bisa menggunakan implementasi anda sendiri dengan **RandomUtils.Random**.

Untuk IOS, saya belum mengimplementasikannya, dan anda harus membuat implementasi **IRandom.**

Di komputer, untuk bisa memperoleh keacakan \(random\) cukup sulit. Masalah terbesarnya adalah, bahwa tidak mungkin untuk bisa mengetahui apakah seri angka tersebut benar-benar acak.

Jika ada sebuah malware berhasil memodifikasi PRNG anda \(maka, ia bisa memprediksi angka-angka yang nantinya akan di generate\), anda tidak bisa melihat hal itu, hingga menyadari ternyata sudah terlambat.

Artinya, implementasi PRNG _cross platform_ \(seperti menggunakan jam komputer yang digabungkan dengan kecepatan CPU\) cukup berbahaya. Karena anda tidak bisa melihatnya.

untuk alasan kinerja, kebanyakan PRNG bekerja dengan cara yang sama: serangkaian angka acak, yang disebut dengan **Seed**, setelah dipilih, lalu formula yang terprediksi itu menggenerate angka berikutnya setiap kali anda membutuhkannya.

Jumlah keacakan _seed_ ditentukan oleh sebuah ukuran yang disebut dengan **Entropy**, namun jumlah dari **Entropy** juga bergantung pada pengamat \(observer\).

Katakanlah anda generate sebuah seed dari waktu jam anda.   
bayangkan saja jika jam anda mempunyai resolusi 1ms. \(kenyataannya ternyata lebih dari ~15ms\)

Jika penyerang mengetahui bahwa anda membuat key seminggu lalu, lalu seed anda memiliki  
1000 \* 60 \* 60 \* 24 \* 7 = 604800000 kemungkinan.

Penyerang tersebut, dengan entropy LOG\(604800000;2\) = 29.17 bits.

Memecahkan hal itu pada sebuah komputer membutuhkan waktu kurang dari 2 detik. …Kita menyebutnya dengan “brute forcing”.

Bisa dikatakan, menggunakan waktu + proses id untuk generate seed.  
Mari kita bayangkan kalau ada 1024 yang berbeda id proses.

Jadi sekarang, penyerangnya membutuhkan 604800000 \* 1024 kemungkinan, waktunya kurang lebih 2000 detik.  
Sekarang, mari kita tambahkan waktu ketika saya menyalakan komputer, dengan asumsi, penyerang mengetahui saat saya menyalakan komputer, ia lalu menambahkan 86400000 kemungkinan.

Penyerang harus memecahkan 604800000 \* 1024 \* 86400000 = 5,35088E+19 kemungkinan.  
Namun, perlu diingat bahwa jika penyerang menyusup ke komputer saya, dia bisa mendapatkan potongan info terakhir itu, lalu menurunkan jumlah kemungkinan, dengan mengurangi entropi.

Entropy diukur dengan **LOG\(possibilities;2\)** dan LOG\(5,35088E+19; 2\) = 65 bits.

Apakah cukup? Mungkin saja. Dengan asumsi, penyerang tidak mengetahui info yang berkaitan dengan kemungkinan nyata yang ada.

Tapi karena hash public key terdiri dari 20 bytes = 160 bits, jumlahnya lebih kecil dari total address yang ada di alam semesta. Sehingga mungkin lebih baik.

> **Catatan:** Menambahkan entropy secara linier lebih sulit, dan cracking entropy secara exponensial menjadi lebih sulit.

Cara yang menarik untuk menghasilkan entropi dengan cepat adalah dengan menggunakan tangan anda. Maksudnya adalah dengan menggerakkan mouse.

Jika anda tidak percaya pada platform PRNG sepenuhnya \(agar tidak menjadi [paranoid](http://android-developers.blogspot.fr/2013/08/some-securerandom-thoughts.html)\), anda bisa menambahkan entropi pada output PRNG yang digunakan di NBitcoin.

```cs
RandomUtils.AddEntropy("hello");
RandomUtils.AddEntropy(new byte[] { 1, 2, 3 });
var nsaProofKey = new Key();
```

Pada NBitcoin, saat anda meminta **AddEntropy\(data\)** adalah:  
**additionalEntropy = SHA\(SHA\(data\) ^ additionalEntropy\)**

Lalu jika anda generate angka baru adalah:  
**result = SHA\(PRNG\(\) ^ additionalEntropy\)**

## Key Derivation **Function \(KDF\)** {#key-derivation-function}

Bagaimanapun, yang paling penting bukanlah besarnya jumlah kemungkinan. Karena ini adalah soal berapa lama waktu yang dibutuhkan seorang penyerang agar ia berhasil memecahkan key anda. Disinilah letak peranan KDF.

KDF, atau **Key Derivation Function** adalah cara yang terbaik untuk mempunyai sebuah key yang kuat, meski jika anda memiliki entropy rendah.

Bayangkan jika anda ingin generate sebuah _seed_, lalu penyerang tahu bahwa ada 10.000.000 kemungkinan.  
_Seed_ tersebut biasanya cukup rentan.

Namun bagaimana jika anda bisa membuat cracking tersebut menjadi lebih lambat?  
KDF adalah sebuah fungsi hash yang membuang resource computing sesuai yang diinginkan.  
Contohnya seperti ini:

```cs
var derived = SCrypt.BitcoinComputeDerivedKey("hello", new byte[] { 1, 2, 3 });
RandomUtils.AddEntropy(derived);
```

Meski jika penyerang mengetahui bahwa entropi anda terdiri dari 5 huruf, dia masih harus menjalankan Scrypt untuk memeriksa kemungkinannya, kurang lebih membutuhkan waktu 5 detik di komputer saya.

Keseluruhan gambaran ini adalah: Tidak perlu paranoid dan curiga pada PRNG, karena anda dapat mengurangi serangan dengan menambahkan entropi dan juga menggunakan KDF.  
Namun perlu diingat, bahwa penyerang juga bisa menurunkan entropi itu dengan mengumpulkan informasi yang berkaitan dengan anda dan sistem anda.   
Jika anda menggunakan timestamp sebagai sumber entropi, lalu dia bisa menurunkan entropi itu dengan cara mencoba mengetahui bahwa anda generate key itu seminggu lalu, dan bahwa anda hanya menggunakan komputer anda pada pukul 9am dan 6pm.

Di bagian sebelumnya saya menyinggung tentang KDF, disebut dengan **Scrypt.** Seperti yang saya katakan, tujuan utama dari KDF adalah berusaha upaya _brute force_ itu haruslah membutuhkan biaya yang mahal. Sehingga cukup sulit baginya untuk dapat melakukan serangan itu.

Jadi anda seharusnya tidak akan terkejut jika standar yang digunakan untuk mengenkripsi private key anda menggunakan KDF. Berikut yang tertera pada [BIP38](http://www.codeproject.com/Articles/775226/NBitcoin-Cryptography-Part).

![](../assets/EncryptedKey.png)

```cs
var privateKey = new Key();
var bitcoinPrivateKey = privateKey.GetWif(Network.Main);
Console.WriteLine(bitcoinPrivateKey); // L1tZPQt7HHj5V49YtYAMSbAmwN9zRjajgXQt9gGtXhNZbcwbZk2r
BitcoinEncryptedSecret encryptedBitcoinPrivateKey = bitcoinPrivateKey.Encrypt("password");
Console.WriteLine(encryptedBitcoinPrivateKey); // 6PYKYQQgx947Be41aHGypBhK6TA5Xhi9TdPBkatV3fHbbKrdDoBoXFCyLK
var decryptedBitcoinPrivateKey = encryptedBitcoinPrivateKey.GetSecret("password");
Console.WriteLine(decryptedBitcoinPrivateKey); // L1tZPQt7HHj5V49YtYAMSbAmwN9zRjajgXQt9gGtXhNZbcwbZk2r

Console.ReadLine();
```

Enkripsi tersebut digunakan pada dua kasus yang berbeda:

* Jika anda tidak percaya untuk menggunakan provider ruang penyimpanan \(karena bisa di hacked\)  
* Anda menyimpan key atas nama orang lain \(dan anda tidak ingin mengetahui key miliknya\)  

Jika anda mempunyai ruang penyimpanan sendiri, maka level enkripsi database saja mungkin sudah cukup.

Namun anda perlu berhati-hati jika server anda menangani _decrypting_ key, karena penyerang mungkin bisa melakukan serangan DDOS kepada server anda, dan memaksa decrypt banyak key.

## Hal yang baik sepanjang waktu {#like-the-good-ol-days}

Pertama, mengapa harus generati beberapa key?  
Alasan utama adalah tentang privasi. Karena anda bisa melihat balance pada seluruh address, maka sebaiknya anda menggunakan address baru di setiap transaksi.

Namun dalam prakteknya, anda memang juga dapat generate key untuk setiap kontak. Karena ini adalah cara yang termudah untuk mengidentifikasi para pembayar, tanpa harus membocorkan banyak privasi anda.

Anda bisa generate key, seperti yang telah anda lakukan di bagian awal:

```cs
var privateKey = new Key()
```

Namun nantinya akan ada dua masalah yang dihadapi jika kita ingin dapat generate address baru pada setiap transaksi:

* Semua backup wallet yang anda miliki menjadi usang saat anda generate key baru.  
* Anda tidak bisa mendelegasikan proses pembuatan address pada rekan yang tidak bisa dipercaya _\(untrusted peer\)_.  

JIka anda sedang mengembangkan sebuah web wallet dan generate key atas nama pengguna, jika satu pengguna di hacked, maka dia akan mencurigai anda.

## BIP38 \(Bagian 2\) {#bip38-part-2}

Kita sudah melihat tentang BIP38 untuk encrypt sebuah key, namun sebetulnya pada BIP ini, terdapat dua ide dalam satu dokumen.

Bagian kedua dari BIP ini, menunjukkan bagaimana anda bisa mendelegasikan Key dan pembuatan Address untuk _untrusted peer_. Hal ini dapat memperbaiki salah satu hal yang menjadi perhatian kami.

**Idenya adalah untuk dapat generate sebuah PassphraseCode pada key generator. Dengan PassphraseCode ini, akan mampu untuk generate keys yang terenkripsi atas nama anda, tanpa harus mengetahui password, ataupun juga private key anda. **

**PassphraseCode** ini, dapat berupa format WIF.

> **Tips**: Dalam NBitcoin, semua jenis prefix dari “Bitcoin” adalah Base58 \(WIF\).

Jadi, sebagai pengguna yang ingin mendelegasikan pembuatan key, anda terlebih dahulu membuat **PassphraseCode**.

![](../assets/PassphraseCode.png)

```cs
var passphraseCode = new BitcoinPassphraseCode("my secret", Network.Main, null);
```

**Lalu memberikan passphraseCode ini kepada** **key generator** **dari pihak ketiga \(third party\).**

Kemudian oleh third party tersebut, akan generate key baru yang telah terenkripsi kepada anda.

![](../assets/PassphraseCodeToEncryptedKeys.png)

```cs
EncryptedKeyResult encryptedKeyResult = passphraseCode.GenerateEncryptedSecret();
```

Pada **EncryptedKeyResult **terdapat banyak informasi:

![](../assets/EncryptedKeyResult.png)

Pertama: **generated address** **bitcoin**,

```cs
var generatedAddress = encryptedKeyResult.GeneratedAddress; // 14KZsAVLwafhttaykXxCZt95HqadPXuz73
```

Lalu **EncryptedKey** itu sendiri, \(seperti yang kita lihat sebelumnya, tentang **enkripsi key**\),

```cs
var encryptedKey = encryptedKeyResult.EncryptedKey; // 6PnWtBokjVKMjuSQit1h1Ph6rLMSFz2n4u3bjPJH1JMcp1WHqVSfr5ebNS
```

dan selanjutnya, **ConfirmationCode**, jadi pihak ketiga dapat membuktikan bahwa key yang digenerate dan address yang sesuai akan cukup efektif bagi password anda.

```cs
var confirmationCode = encryptedKeyResult.ConfirmationCode; // cfrm38VUcrdt2zf1dCgf4e8gPNJJxnhJSdxYg6STRAEs7QuAuLJmT5W7uNqj88hzh9bBnU9GFkN
```

Sebagai pemilik, anda juga akan menerima informasi ini. Anda perlu memeriksa key generator itu tidak menipu anda dengan menggunakan **ConfirmationCode.Check**, lalu ambil private key anda menggunakan password:

```cs
Console.WriteLine(confirmationCode.Check("my secret", generatedAddress)); // True
var bitcoinPrivateKey = encryptedKey.GetSecret("my secret");
Console.WriteLine(bitcoinPrivateKey.GetAddress() == generatedAddress); // True
Console.WriteLine(bitcoinPrivateKey); // KzzHhrkr39a7upeqHzYNNeJuaf1SVDBpxdFDuMvFKbFhcBytDF1R
```

Jadi, sekarang anda telah melihat bagaimana third party dapat melakukan generate key terenkripsi atas nama anda, tanpa harus mengetahui password dan private key anda.

![](../assets/ThirdPartyKeyGeneration.png)

Tapi ingat, masih ada satu masalah tersisa:

* Semua backup wallet anda akan jadi usang saat anda generate key baru.

BIP 32, atau Hierarchical Deterministic Wallets \(wallet HD\) dapat menjadi solusi, dan cukup support secara luas.

## HD Wallet \(BIP 32\) {#hd-wallet-bip-32}

Tetaplah mengingat tentang masalah-masalah yang ingin kita pecahkan:

* Menjaga agar backup tidak usang
* Delegasi key \/ address kepada untrusted peer

Sebuah wallet “Deterministic” akan menangani masalah backup. Dengan wallet semacam ini, anda cukup hanya menyimpan seed saja. Dari seed tersebut, anda bisa generate seri private key yang sama berulang kali.

Itu adalah kegunaan wallet “Deterministic”.  
Seperti yang bisa anda lihat, dari master key, saya bisa generate key baru:

```cs
ExtKey masterKey = new ExtKey();
Console.WriteLine("Master key : " + masterKey.ToString(Network.Main));
for (int i = 0; i < 5; i++)
{
    ExtKey key = masterKey.Derive((uint)i);
    Console.WriteLine("Key " + i + " : " + key.ToString(Network.Main));
}
```

```
Master key : xprv9s21ZrQH143K3JneCAiVkz46BsJ4jUdH8C16DccAgMVfy2yY5L8A4XqTvZqCiKXhNWFZXdLH6VbsCsqBFsSXahfnLajiB6ir46RxgdkNsFk
Key 0 : xprv9tvBA4Kt8UTuEW9Fiuy1PXPWWGch1cyzd1HSAz6oQ1gcirnBrDxLt8qsis6vpNwmSVtLZXWgHbqff9rVeAErb2swwzky82462r6bWZAW6Ty
Key 1 : xprv9tvBA4Kt8UTuHyzrhkRWh9xTavFtYoWhZTopNHGJSe3KomssRrQ9MTAhVWKFp4d7D8CgmT7TRzauoAZXp3xwHQfxr7FpXfJKpPDUtiLdmcF
Key 2 : xprv9tvBA4Kt8UTuLoEZPpW9fBEzC3gfTdj6QzMp8DzMbAeXgDHhSMmdnxSFHCQXycFu8FcqTJRm2kamjeE8CCKzbiXyoKWZ9ihiF7J5JicgaLU
Key 3 : xprv9tvBA4Kt8UTuPwJQyxuZoFj9hcEMCoz7DAWLkz9tRMwnBDiZghWePdD7etfi9RpWEWQjKCM8wHvKQwQ4uiGk8XhdKybzB8n2RVuruQ97Vna
Key 4 : xprv9tvBA4Kt8UTuQoh1dQeJTXsmmTFwCqi4RXWdjBp114rJjNtPBHjxAckQp3yeEFw7Gf4gpnbwQTgDpGtQgcN59E71D2V97RRDtxeJ4rVkw4E
Key 5 : xprv9tvBA4Kt8UTuTdiEhN8iVDr5rfAPSVsCKpDia4GtEsb87eHr8yRVveRhkeLEMvo3XWL3GjzZvncfWVKnKLWUMNqSgdxoNm7zDzzD63dxGsm
```

Anda hanya cukup menyimpan saja **masterKey**, karena anda dapat generate private key yang sama berulang kali.

Seperti yang bisa dilihat disini, key ini adalah **ExtKey** dan bukan **Key** yang biasa anda gunakan. Namun, ini tidak menjadi penghalang karena anda mempunyai private key yang sebenarnya di dalamnya:

![](../assets/ExtKey.png)

Anda bisa kembali pada sebuah **Key** ke **ExtKey** dengan memberi **Key** lalu **ChainCode** kepada kontruksi **ExtKey**. dan ini akan dapat bekerja seperti dibawah ini:

```cs
ExtKey extKey = new ExtKey();
byte[] chainCode = extKey.ChainCode;
Key key = extKey.PrivateKey;

ExtKey newExtKey = new ExtKey(key, chainCode);
```

Type **base58** sama dengan **ExtKey,** disebut dengan **BitcoinExtKey**.

Namun bagaimana cara kita menyelesaikan masalah kedua? untuk mendelegasikan pembuatan address kepada peer, yang sebenarnya cukup berpotensi dapat di hacked \(seperti pada server payment\)?

Triknya, anda bisa melakukan “neuter” dari master key anda, lalu anda mempunyai versi public key \(tanpa private key\) dari master key. Pada versi "neuter" ini, pihak ketiga dapat generate public key tanpa mengetahui private key.

```cs
ExtPubKey masterPubKey = masterKey.Neuter();
for (int i = 0 ; i < 5 ; i++)
{
    ExtPubKey pubkey = masterPubKey.Derive((uint)i);
    Console.WriteLine("PubKey " + i + " : " + pubkey.ToString(Network.Main));
}
```

\`\`\`PubKey 0 : xpub67uQd5a6WCY6A7NZfi7yGoGLwXCTX5R7QQfMag8z1RMGoX1skbXAeB9JtkaTiDoeZPprGH1drvgYcviXKppXtEGSVwmmx4pAdisKv2CqoWS
PubKey 1 : xpub67uQd5a6WCY6CUeDMBvPX6QhGMoMMNKhEzt66hrH6sv7rxujt7igGf9AavEdLB73ZL6ZRJTRnhyc4BTiWeXQZFu7kyjwtDg9tjRcTZunfeR
PubKey 2 : xpub67uQd5a6WCY6Dxbqk9Jo9iopKZUqg8pU1bWXbnesppsR3Nem8y4CVFjKnzBUkSVLGK4defHzKZ3jjAqSzGAKoV2YH4agCAEzzqKzeUaWJMW
PubKey 3 : xpub67uQd5a6WCY6HQKya2Mwwb7bpSNB5XhWCR76kRaPxchE3Y1Y2MAiSjhRGftmeWyX8cJ3kL7LisJ3s4hHDWvhw3DWpEtkihPpofP3dAngh5M
PubKey 4 : xpub67uQd5a6WCY6JddPfiPKdrR49KYEuXUwwJJsL5rWGDDQkpPctdkrwMhXgQ2zWopsSV7buz61e5mGSYgDisqA3D5vyvMtKYP8S3EiBn5c1u4

    So imagine that your payment server generate pubkey1, you can get the corresponding private key with your private master key.

    ```cs
    masterKey = new ExtKey();
    masterPubKey = masterKey.Neuter();

    //The payment server generate pubkey1
    ExtPubKey pubkey1 = masterPubKey.Derive((uint)1);

    //You get the private key of pubkey1
    ExtKey key1 = masterKey.Derive((uint)1);

    //Check it is legit
    Console.WriteLine("Generated address : " + pubkey1.PubKey.GetAddress(Network.Main));
    Console.WriteLine("Expected address : " + key1.PrivateKey.PubKey.GetAddress(Network.Main));

```
Generated address : 1Jy8nALZNqpf4rFN9TWG2qXapZUBvquFfX
Expected address : 1Jy8nALZNqpf4rFN9TWG2qXapZUBvquFfX
```

**ExtPubKey** mirip dengan **ExtKey** kecuali karena menyimpan **PubKey** dan bukan sebuah **Key**.

![](../assets/ExtPubKey.png)

Anda telah melihat bagaimana key Deterministic dapat menyelesaikan persoalan kita. Mari sekarang kita bicara apa kegunaan dari “hierarchical”.

Pada latihan sebelumnya, anda telah mengetahui bagaimana menggabungkan master key + index yang memungkinkan kita dapat key lain. Kita menyebut proses ini dengan proses **Derivation**, master key ini adalah **parent key**, sedangkan key hasil generated disebut dengan **child key**.

Namun, anda juga dapat derivate _children_ dari child key. Dan ini menjadi fungsi dari “hierarchical”.

Berikut adalah konseptual proses yang berlangsung secara umum: **Parent Key + KeyPath =&gt; Child Key**

![](../assets/Derive1.png)

![](../assets/Derive2.png)

Pada diagram tersebut, anda dapat derivate Child\(1,1\) dari _parent_ dengan dua cara yang berbeda:

```cs
ExtKey parent = new ExtKey();
ExtKey child11 = parent.Derive(1).Derive(1);
```

Atau

```cs
ExtKey parent = new ExtKey();
ExtKey child11 = parent.Derive(new KeyPath("1/1"));
```

Jadi kesimpulannya:

![](../assets/DeriveKeyPath.png)

Dan itu juga bekerja pada **ExtPubKey**.

Mengapa anda membutuhkan key hierarchical? Karena akan lebih bagus untuk mengklasifikasikan tipe key untuk multi account. Lebih jauh bisa dilihat di [BIP44](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki).

Bisa juga berguna untuk membagi hak akses per segmen account di sebuah organisasi.

Misalnya saja anda adalah CEO di sebuah perusahaan. Anda ingin dapat menangani seluruh wallet di perusahaan, namun anda tidak ingin departemen akutansi mengambil dana dari departemen marketing.

Jadi ide pertama yang dilakukan adalah dengan generate satu hierarchy untuk tiap departemen.

![](../assets/CeoMarketingAccounting.png)

Namun, dalam kasus tertentu, departemen **Akutansi** dan **Marketing** mungkin dapat _recover_ private key milik CEO, jika sewaktu-waktu dibutuhkan.

Maka kita dapat menentukkan child key sebagai **non-hardened**.

![](../assets/NonHardened.png)

```cs
ExtKey ceoKey = new ExtKey();
Console.WriteLine("CEO: " + ceoKey.ToString(Network.Main));
ExtKey accountingKey = ceoKey.Derive(0, hardened: false);

ExtPubKey ceoPubkey = ceoKey.Neuter();

//Recover ceo key with accounting private key and ceo public key
ExtKey ceoKeyRecovered = accountingKey.GetParentExtKey(ceoPubkey);
Console.WriteLine("CEO recovered: " + ceoKeyRecovered.ToString(Network.Main));
```

```
CEO: xprv9s21ZrQH143K2XcJU89thgkBehaMqvcj4A6JFxwPs6ZzGYHYT8dTchd87TC4NHSwvDuexuFVFpYaAt3gztYtZyXmy2hCVyVyxumdxfDBpoC
CEO recovered: xprv9s21ZrQH143K2XcJU89thgkBehaMqvcj4A6JFxwPs6ZzGYHYT8dTchd87TC4NHSwvDuexuFVFpYaAt3gztYtZyXmy2hCVyVyxumdxfDBpoC
```

Dilain kata, **non-hardened key** dapat “naik” hirarkinya. **Non-hardened key** hanya dapat digunakan untuk memberikan kategori account yang mempunyai **single control **\(kontrol tunggal\).

Dalam hal ini, CEO harus membuat sebuah **hardened key**, sehingga departemen akutansi tidak dapat naik hirarkinya.

```cs
ExtKey ceoKey = new ExtKey();
Console.WriteLine("CEO: " + ceoKey.ToString(Network.Main));
ExtKey accountingKey = ceoKey.Derive(0, hardened: true);

ExtPubKey ceoPubkey = ceoKey.Neuter();

ExtKey ceoKeyRecovered = accountingKey.GetParentExtKey(ceoPubkey); //Crash
```

Anda juga dapat membuat hardened key melalui **ExtKey.Derivate**\(**KeyPath\)**, dengan kebalikannya, setelah index child:

```cs
var nonHardened = new KeyPath("1/2/3");
var hardened = new KeyPath("1/2/3'");
```

Jadi mari kita bayangkan jika departemen akutansi dapat generate 1 parent key untuk setiap customer, dan sebuah child untuk setiap pembayaran customer.

Sebagai seorang CEO, anda mungkin ingin menggunakan dana pada salah satu address. Jadi begini prosesnya:

```cs
ceoKey = new ExtKey();
string accounting = "1'";
int customerId = 5;
int paymentId = 50;
KeyPath path = new KeyPath(accounting + "/" + customerId + "/" + paymentId);
//Path : "1'/5/50"
ExtKey paymentKey = ceoKey.Derive(path);
```

## Kode Mnemonic Untuk Key HD \(BIP39\) {#mnemonic-code-for-hd-keys-bip39}

Setelah melihatnya sendiri, generate sebuah key HD cukup mudah. Namun, jika kita ingin cara yang termudah untuk dapat mentransmit key dengan sebuah telephone atau _hand writing_?

Pada sebuah **_Cold wallets_** seperti **Trezor**, dapat generate key HD Keys dari sebuah kalimat yang bisa secara mudah dituliskan kembali. Mereka meyebut kalimat ini dengan “_the seed_” atau “_mnemonic_”. Dan itu di proteksi menggunakan sebuah password atau sebuah PIN.  
![](../assets/Trezor.png)

Bahasa yang anda gunakan untuk generate kalimat yang mudah untuk dituliskan ini disebut dengan **Wordlist**

![](../assets/RootKey.png)

```cs
Mnemonic mnemo = new Mnemonic(Wordlist.English, WordCount.Twelve);
ExtKey hdRoot = mnemo.DeriveExtKey("my password");
Console.WriteLine(mnemo);
```

`minute put grant neglect anxiety case globe win famous correct turn link`

Sekarang, jika anda telah mempunyai _mnemonic_ dan _password_, maka anda dapat merecover key **hdRoot**.

```cs
mnemo = new Mnemonic("minute put grant neglect anxiety case globe win famous correct turn link",
                Wordlist.English);
hdRoot = mnemo.DeriveExtKey("my password");
```

Bahasa yang support untuk **wordlist** adalah, Inggris, Jepang, Spanyol, dan China.

## Dark Wallet {#dark-wallet}

Nama yang digunakan tersebut tentu saja tidak berarti ada kegelapan disana, karena itu hanya dipakai untuk menarik perhatian saja. Dark Wallet dapat menjadi solusi praktis untuk dapat juga menyelesaikan dua persoalan kita diatas:

* Menjaga backup agar tidak usang
* Delegasi key \/ address kepada untrusted peer

Namun juga mempunyai sebuah fitur yang bagus.

Karena anda hanya perlu membagikan satu address saja kepada publik \(disebut dengan **StealthAddress**\), tanpa harus mengumbar privasi anda.

Mari kita ingat kembali, bahwa jika anda membagikan satu **AddressBitcoin** kepada orang lain, lalu mereka bisa melihat balance anda dengan cara melihatnya di blockchain… Namun tidak demikian jika menggunakan **StealthAddress**.

Sebenarnya agak disayangkan karena pada namanya menggunakan label "**dark", **karena pada dasarnya mampu menyelesaikan persoalan penting tentang privasi. Mungkin nama yang sebaiknya digunakan adalah: **One Address**.

Terminologi di dalam Dark Wallet, digambarkan dengan pelaku yang berbeda, seperti pada berikut ini:

* **Payer** _\(pengirim\)_ mengetahui **StealthAddress** **Receiver **_\(penerima\)_
* **Receiver** mengetahui **Spend Key**, sebuah key rahasia yang memungkinkannya untuk mengirim koin yang diterimanya dari sebuah transaksi. 
* **Scanner** mengetahui **Scan Key**, sebuah key rahasia yang memungkinkannya untuk mendeteksi transaksi kepada **Receiver**.

Pada detail sisa proses operasionalnya, **StealthAddress** terdiri dari beberapa **Spend PubKey** \(untuk multi sig\), dan sebuah **Scan PubKey**.

![](../assets/StealthAddress.png)

```cs
var scanKey = new Key();
var spendKey = new Key();
BitcoinStealthAddress stealthAddress
    = new BitcoinStealthAddress
        (
        scanKey: scanKey.PubKey,
        pubKeys: new[] { spendKey.PubKey },
        signatureCount: 1,
        bitfield: null,
        network: Network.Main);
```

**Payer**, akan menggunakan **StealthAddress**, generate key sementara yang disebut dengan **Ephem Key** dan selanjutnya sebuah **Stealth Pub Key**, dari address Bitcoin address yang digunakan untuk pembayaran tersebut.

![](../assets/EphemKey.png)

Lalu, dia akan menaruh **Ephem PubKey** itu dalam sebuah obyek _embedded_ **Stealth Metadata** yang berada di OP\_RETURN pada sebuah transaksi. \(sama seperti yang pernah kita lakukan pada tantangan latihan pertama\)

Dia juga akan menambahkan pada output address bitcoin yang di generate. \(address dari **Stealth pub key**\)

![](../assets/StealthMetadata.png)

```cs
var ephemKey = new Key();
Transaction transaction = new Transaction();
stealthAddress.SendTo(transaction, Money.Coins(1.0m), ephemKey);
Console.WriteLine(transaction);
```

Pembuatan **EphemKey** diimplementasikan seperti pada detail berikut, anda bisa mencobanya, NBitcoin dapat generate secara otomatis:

```cs
Transaction transaction = new Transaction();
stealthAddress.SendTo(transaction, Money.Coins(1.0m));
Console.WriteLine(transaction);
```

```json
{
  "hash": "7772b0ad19acd1bd2b0330238a898fe021486315bd1e15f4154cd3931a4940f9",
  "ver": 1,
  "vin_sz": 0,
  "vout_sz": 2,
  "lock_time": 0,
  "size": 93,
  "in": [],
  "out": [
    {
      "value": "0.00000000",
      "scriptPubKey": "OP_RETURN 060000000002b9266f15e8c6598e7f25d3262969a774df32b9b0b50fea44fc8d914c68176f3e"
    },
    {
      "value": "1.00000000",
      "scriptPubKey": "OP_DUP OP_HASH16051f68af989f5bf24259c519829f46c7f2935b756 OP_EQUALVERIFY OP_CHECKSIG"
    }
  ]
}
```

Lalu payer menambah dan menandatangani input, kemudian mengirim transaksi itu ke dalam jaringan.

**Scanner** mengetahui **StealthAddress** dan **Scan Key, **dapat merecover **Stealth PubKey** dan juga **AddressBitcoin.**

![](../assets/ScannerRecover.png)

Scanner memeriksa apakah satu dari output transaksi sesuai untuk addressnya. Jika sesuai, lalu **Scanner** memberikan notif pada **Receiver** tentang transaksi tersebut.

**Receiver** bisa mendapat private key dari address dengan **Spend Key**.

![](../assets/ReceiverStealth.png)

Dari kode tersebut menjelaskan bagaimana sebuah Scanner, untuk melakukan scan sebuah transaksi, dan sebagai Receiver, untuk mendapat private key, selanjutnya akan dijelaskan lebih jauh di **TransactionBuilder** \(pada bagian Tipe Kepemilikan Lain\).

Perlu dijadikan catatan bahwa sebuah **StealthAddress** dapat memiliki beberapa **spend pubkeys**, dalam hal ini, address merepresentasikan sebuah multi sig.

Keterbatasan dari Dark Wallet adalah dalam penggunaan **OP\_RETURN**, jadi kita tidak dapat dengan begitu mudah untuk bisa _embed_ arbitrary data pada sebuah transaksi, seperti yang pernah kita lakukan dalam transfer sebelumnya. \(aturan bitcoin hanya memperbolehkan satu OP\_RETURN terdiri dari 40 bytes, lalu menjadi 80 byte, di tiap transaksi\)

> \([Stackoverflow](http://bitcoin.stackexchange.com/a/29648/26859)\) As I understand it, the "stealth address" is intended to address a very specific problem. If you wish to solicit payments from the public, say by posting a donation address on your website, then everyone can see on the block chain that all those payments went to you, and perhaps try to track how you spend them.
> 
> With a stealth address, you ask payers to generate a unique address in such a way that you \(using some additional data which is attached to the transaction\) can deduce the corresponding private key. So although you publish a single "stealth address" on your website, the block chain sees all your incoming payments as going to separate addresses and has no way to correlate them. \(Of course, any individual payer knows their payment went to you, and can trace how you spend it, but they don't learn anything about other people's payments to you.\)
> 
> But you can get the same effect another way: just give each payer a unique address. Rather than posting a single public donation address on your website, have a button that generates a new unique address and saves the private key, or selects the next address from a long list of pre-generated addresses \(whose private keys you hold somewhere safe\). Just as before, the payments all go to separate addresses and there is no way to correlate them, nor for one payer to see that other payments went to you.
> 
> So the only difference with stealth addresses is essentially to move the chore of producing a unique address from the server to the client. Indeed, in some ways stealth addresses may be worse, since very few people use them, and if you are known to be one of them, it will be easier to connect stealth transactions with you.
> 
> It doesn't provide "100% anonymity". The fundamental anonymity weakness of Bitcoin remains - that everyone can follow the chain of payments, and if you know something about one transaction or the parties to it, you can deduce something about where those coins came from or where they went.

