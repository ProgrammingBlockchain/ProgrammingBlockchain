## Private key {#private-key}

Private keys seringkali digambarkan dalam bentuk Base58Check, disebut dengan **Bitcoin Secret** \(atau dikenal juga dengan **Wallet Import Format** disingkat **WIF**\), sama seperti Address Bitcoin yang dikodekan juga menggunakan Base58Check.

![](../assets/BitcoinSecret.png)

```cs
Key privateKey = new Key(); // generate a random private key
BitcoinSecret mainNetPrivateKey = privateKey.GetBitcoinSecret(Network.Main);  // get our private key for the mainnet
BitcoinSecret testNetPrivateKey = privateKey.GetBitcoinSecret(Network.TestNet);  // get our private key for the testnet
Console.WriteLine(mainNetPrivateKey); // L5B67zvrndS5c71EjkrTJZ99UaoVbMUAK58GKdQUfYCpAa6jypvn
Console.WriteLine(testNetPrivateKey); // cVY5auviDh8LmYUW8AfafseD6p6uFoZrP7GjS3rzAerpRKE9Wmuz

bool WifIsBitcoinSecret = mainNetPrivateKey == privateKey.GetWif(Network.Main);
Console.WriteLine(WifIsBitcoinSecret); // True
```

Perlu dicatat bahwa sangat mudah untuk melangkah dari **BitcoinSecret** ke private **Key**. Di lain sisi, mustahil untuk melangkah dari Address Bitcoin ke Public Key. Hal itu karena Address Bitcoin berisi hash dari Public Key, tidak Public Key itu sendiri.  
Proses informasi ini dilakukan dengan memeriksa kesamaan antara dua codeblocks dibawah ini:

```cs
Key privateKey = new Key(); // generate a random private key
BitcoinSecret bitcoinPrivateKey = privateKey.GetWif(Network.Main); // L5B67zvrndS5c71EjkrTJZ99UaoVbMUAK58GKdQUfYCpAa6jypvn
Key samePrivateKey = bitcoinPrivateKey.PrivateKey;
```

```cs
PubKey publicKey = privateKey.PubKey;
BitcoinPubKeyAddress bitcoinPubicKey = publicKey.GetAddress(Network.Main); // 1PUYsjwfNmX64wS368ZR5FMouTtUmvtmTY
//PubKey samePublicKey = bitcoinPubicKey.ItIsNotPossible;
```

### Latihan:

1. Generate private key di mainnet dan mencatatnya.
2. Dapatkan address yang sesuai.
3. Kirimkan bitcoin ke address tersebut. Sebanyak yang anda tidak sanggup untuk kehilangan bitcoin itu, sehingga anda akan tetap fokus dan termotivasi untuk dapat mendapatkan bitcoin itu kembali sampai ke tahapan selanjutnya.  

