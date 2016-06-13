## Private key {#private-key}

Private keys are often represented in Base58Check called a **Bitcoin Secret** (also known as **Wallet Import Format** or simply **WIF**), like Bitcoin Addresses.  

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

Note that it is easy to go from **BitcoinSecret** to private **Key**. On the other hand, it is impossible to go from a Bitcoin Address to Public Key because the Bitcoin Address contains a hash of the Public Key, not the Public Key itself.  
Process this information by examining the similarities between these two codeblocks:  

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

### Exercise:
1. Generate a private key on the mainnet and note it.
2. Get the corresponding address.
3. Send bitcoins to it. As much as you cannot afford to lose, so it will keep you focused and motivated to get them back during the following lessons. 


 





