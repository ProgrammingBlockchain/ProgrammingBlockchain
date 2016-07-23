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

then the **EncryptedKey** itself, \(as we have seen in the previous, **Key Encryption** lesson\),

```cs
var encryptedKey = encryptedKeyResult.EncryptedKey; // 6PnWtBokjVKMjuSQit1h1Ph6rLMSFz2n4u3bjPJH1JMcp1WHqVSfr5ebNS
```

and last but not the least, the **ConfirmationCode**, so that the third party can prove that the generated key and address correspond effectively to your password.

```cs
var confirmationCode = encryptedKeyResult.ConfirmationCode; // cfrm38VUcrdt2zf1dCgf4e8gPNJJxnhJSdxYg6STRAEs7QuAuLJmT5W7uNqj88hzh9bBnU9GFkN
```

As the owner, once you receive these info, you need to check that the key generator did not cheat by using **ConfirmationCode.Check**, then get your private key with your password:

```cs
Console.WriteLine(confirmationCode.Check("my secret", generatedAddress)); // True
var bitcoinPrivateKey = encryptedKey.GetSecret("my secret");
Console.WriteLine(bitcoinPrivateKey.GetAddress() == generatedAddress); // True
Console.WriteLine(bitcoinPrivateKey); // KzzHhrkr39a7upeqHzYNNeJuaf1SVDBpxdFDuMvFKbFhcBytDF1R
```

So, we have just seen how the third party can generate encrypted key on your behalf, without knowing your password and private key.

![](../assets/ThirdPartyKeyGeneration.png)

However, one problem remains:

* All backup of your wallet that you have will become outdated when you generate a new key.

BIP 32, or Hierarchical Deterministic Wallets \(HD wallets\) proposes another solution, and is more widely supported.

## HD Wallet \(BIP 32\) {#hd-wallet-bip-32}

Let’s keep in mind the problems that we want to resolve:

* Prevent outdated backups
* Delegating key \/ address generation to an untrusted peer

A “Deterministic” wallet would fix our backup problem. With such wallet, you would have to save only the seed. From this seed, you can generate the same series of private key over and over.

This is what the “Deterministic” stands for.  
As you can see, from the master key, I can generate new keys:

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

You only need to save the **masterKey**, since you can generate the same suite of private keys over and over.

As you can see, these keys are **ExtKey** and not **Key** as you are used to. However, this should not stop you since you have the real private key inside:

![](../assets/ExtKey.png)

You can go back from a **Key** to an **ExtKey** by supplying the **Key** and the **ChainCode** to the **ExtKey** constructor. This works as follows:

```cs
ExtKey extKey = new ExtKey();
byte[] chainCode = extKey.ChainCode;
Key key = extKey.PrivateKey;

ExtKey newExtKey = new ExtKey(key, chainCode);
```

The **base58** type equivalent of **ExtKey** is called **BitcoinExtKey**.

But how can we solve our second problem: delegating address creation to a peer that can potentially be hacked \(like a payment server\)?

The trick is that you can “neuter” your master key, then you have a public \(without private key\) version of the master key. From this neutered version, a third party can generate public keys without knowing the private key.

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

**ExtPubKey** is similar to **ExtKey** except that it holds a **PubKey** and not a **Key**.

![](../assets/ExtPubKey.png)

Now we have seen how Deterministic keys solve our problems, let’s speak about what the “hierarchical” is for.

In the previous exercise, we have seen that by combining master key + index we could generate another key. We call this process **Derivation**, master key is the **parent key**, and the generated key is called **child key**.

However, you can also derivate children from the child key. This is what the “hierarchical” stands for.

This is why conceptually more generally you can say: Parent Key + KeyPath =&gt; Child Key

![](../assets/Derive1.png)

![](../assets/Derive2.png)

In this diagram, you can derivate Child\(1,1\) from parent in two different way:

```cs
ExtKey parent = new ExtKey();
ExtKey child11 = parent.Derive(1).Derive(1);
```

Or

```cs
ExtKey parent = new ExtKey();
ExtKey child11 = parent.Derive(new KeyPath("1/1"));
```

So in summary:

![](../assets/DeriveKeyPath.png)

It works the same for **ExtPubKey**.

Why do you need hierarchical keys? Because it might be a nice way to classify the type of your keys for multi account purpose. More on [BIP44](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki).

It also permit to segment account rights across an organization.

Imagine you are CEO of a company. You want control over all wallet, but you don’t want that the Accounting department spend the money of the Marketing department.

So your first idea would be to generate one hierarchy for each department.

![](../assets/CeoMarketingAccounting.png)

However, in such case, **Accounting** and **Marketing** would be able to recover the CEO’s private key.

We define such child keys as **non-hardened**.

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

In other words, a **non-hardened key** can “climb” the hierarchy.**Non-hardened key** should only be used for categorizing accounts that belongs to a **single control**.

So in our case, the CEO should create a **hardened key**, so the accounting department will not be able to climb.

```cs
ExtKey ceoKey = new ExtKey();
Console.WriteLine("CEO: " + ceoKey.ToString(Network.Main));
ExtKey accountingKey = ceoKey.Derive(0, hardened: true);

ExtPubKey ceoPubkey = ceoKey.Neuter();

ExtKey ceoKeyRecovered = accountingKey.GetParentExtKey(ceoPubkey); //Crash
```

You can also create hardened key by via the **ExtKey.Derivate**\(**KeyPath\)**, by using an apostrophe after a child’s index:

```cs
var nonHardened = new KeyPath("1/2/3");
var hardened = new KeyPath("1/2/3'");
```

So let’s imagine that the Accounting Department generate 1 parent key for each customer, and a child for each of the customer’s payment.

As the CEO, you want to spend the money on one of these addresses. Here is how you would proceed.

```cs
ceoKey = new ExtKey();
string accounting = "1'";
int customerId = 5;
int paymentId = 50;
KeyPath path = new KeyPath(accounting + "/" + customerId + "/" + paymentId);
//Path : "1'/5/50"
ExtKey paymentKey = ceoKey.Derive(path);
```

## Mnemonic Code for HD Keys \(BIP39\) {#mnemonic-code-for-hd-keys-bip39}

As you have seen, generating an HD keys is easy. However, what if we want as easy way to transmit such key by telephone or hand writing?

Cold wallets like Trezor, generates the HD Keys from a sentence that can easily be written down. They call such sentence “the seed” or “mnemonic”. And it can eventually be protected by a password or a PIN.  
![](../assets/Trezor.png)

The language that you use to generate your easy to write sentence is called a **Wordlist**

![](../assets/RootKey.png)

```cs
Mnemonic mnemo = new Mnemonic(Wordlist.English, WordCount.Twelve);
ExtKey hdRoot = mnemo.DeriveExtKey("my password");
Console.WriteLine(mnemo);
```

`minute put grant neglect anxiety case globe win famous correct turn link`

Now, if you have the mnemonic and the password, you can recover the **hdRoot** key.

```cs
mnemo = new Mnemonic("minute put grant neglect anxiety case globe win famous correct turn link",
                Wordlist.English);
hdRoot = mnemo.DeriveExtKey("my password");
```

Currently supported **wordlist** are, English, Japanese, Spanish, Chinese \(simplified and traditional\).

## Dark Wallet {#dark-wallet}

This name is unfortunate since there is nothing dark about it, and it attract unwanted attention and worries.Dark Wallet is a practical solution that fix our two initial problems:

* Prevent outdated backups
* Delegating key \/ address generation to an untrusted peer

But it has a bonus killer feature.

You have to share only one address with the world \(called **StealthAddress**\), without leaking any privacy.

Let’s remind us that if you share one **BitcoinAddress** with everybody, then all can see your balance by consulting the blockchain… That’s not the case with a **StealthAddress**.

This is a real shame it was labeled as **dark** since it solves partially the important problem of privacy leaking caused by the pseudo-anonymity of Bitcoin. A better name would have been: **One Address**.

In Dark Wallet terminology, here are the different actors:

* The **Payer** knows the **StealthAddress** of the **Receiver**
* The **Receiver** knows the **Spend Key**, a secret that will allow him to spend the coins he receives from one of such transaction.
* **Scanner** knows the **Scan Key**, a secret that allows him to detect the transactions those belong to the **Receiver**.

The rest is operational details.Underneath, this **StealthAddress** is composed of one or several **Spend PubKey** \(for multi sig\), and one **Scan PubKey**.

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

The **payer**, will take your **StealthAddress**, generate a temporary key called **Ephem Key** and will generate a **Stealth Pub Key**, from which the Bitcoin address to which the payment will be done is generated.

![](../assets/EphemKey.png)

Then, he will package the **Ephem PubKey** in a **Stealth Metadata** object embedded that in the OP\_RETURN of the transaction \(as we have done for the first challenge\)

He will also add the output to the generated bitcoin address. \(the address of the **Stealth pub key**\)

![](../assets/StealthMetadata.png)

```cs
var ephemKey = new Key();
Transaction transaction = new Transaction();
stealthAddress.SendTo(transaction, Money.Coins(1.0m), ephemKey);
Console.WriteLine(transaction);
```

The creation of the **EphemKey** being an implementation details, you can omit it, NBitcoin will generate one automatically:

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

Then the payer add and signs the inputs, then sends the transaction on the network.

The **Scanner** knowing the **StealthAddress** and the **Scan Key** can recover the **Stealth PubKey** and so expected **BitcoinAddress** payment.

![](../assets/ScannerRecover.png)

Then the scanner checks if one of the output of the transaction correspond to such address. If it is, then **Scanner** notify the **Receiver** about the transaction.

The **Receiver** can then get the private key of the address with his **Spend Key**.

![](../assets/ReceiverStealth.png)

The code explaining how, as a Scanner, to scan a transaction and how, as a Receiver, to uncover the private key, will be explained later in the **TransactionBuilder** \(Other types of ownership\) part.

It should be noted that a **StealthAddress** can have multiple **spend pubkeys**, in which case, the address represent a multi sig.

One limit of Dark Wallet is the use of **OP\_RETURN**, so we can’t easily embed arbitrary data in the transaction as we have done for in Bitcoin Transfer. \(Current bitcoin rules allows only one OP\_RETURN of 40 bytes, soon 80, per transaction\)

> \([Stackoverflow](http://bitcoin.stackexchange.com/a/29648/26859)\) As I understand it, the "stealth address" is intended to address a very specific problem. If you wish to solicit payments from the public, say by posting a donation address on your website, then everyone can see on the block chain that all those payments went to you, and perhaps try to track how you spend them.
> 
> With a stealth address, you ask payers to generate a unique address in such a way that you \(using some additional data which is attached to the transaction\) can deduce the corresponding private key. So although you publish a single "stealth address" on your website, the block chain sees all your incoming payments as going to separate addresses and has no way to correlate them. \(Of course, any individual payer knows their payment went to you, and can trace how you spend it, but they don't learn anything about other people's payments to you.\)
> 
> But you can get the same effect another way: just give each payer a unique address. Rather than posting a single public donation address on your website, have a button that generates a new unique address and saves the private key, or selects the next address from a long list of pre-generated addresses \(whose private keys you hold somewhere safe\). Just as before, the payments all go to separate addresses and there is no way to correlate them, nor for one payer to see that other payments went to you.
> 
> So the only difference with stealth addresses is essentially to move the chore of producing a unique address from the server to the client. Indeed, in some ways stealth addresses may be worse, since very few people use them, and if you are known to be one of them, it will be easier to connect stealth transactions with you.
> 
> It doesn't provide "100% anonymity". The fundamental anonymity weakness of Bitcoin remains - that everyone can follow the chain of payments, and if you know something about one transaction or the parties to it, you can deduce something about where those coins came from or where they went.

