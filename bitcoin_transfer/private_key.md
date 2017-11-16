## Private key {#private-key}

Private keys are often represented in Base58Check encoding scheme. And a private key represented in Base58Check is called a **Bitcoin Secret**. Bitcoin secret is also known as a **Wallet Import Format** or simply **WIF**, because a Bitcoin secret is usually used with a Bitcoin wallet which is a kind of an UI tool, along with a Bitcoin address. Usually a Bitcoin secret is used for signing a signature and proof of ownership for the coin, and a Bitcoin address is used for representing a recipient to which the coin will be sent, both for the UI for a Bitcoin transaction.

![](../assets/BitcoinSecret.png)  

```cs  
//Generate a random key object.
Key privateKey = new Key(); 
//Generate a Bitcoin secret for a MainNet, which is nothing but a private key represented in Base58Check.
BitcoinSecret mainNetPrivateKey = privateKey.GetBitcoinSecret(Network.Main);
//Generate a Bitcoin secret for a TestNet, which is nothing but a private key represented in Base58Check.
BitcoinSecret testNetPrivateKey = privateKey.GetBitcoinSecret(Network.TestNet);

Console.WriteLine($"privateKeyForMainNet: {privateKeyForMainNet}"); 
//Output:
//L5B67zvrndS5c71EjkrTJZ99UaoVbMUAK58GKdQUfYCpAa6jypvn
Console.WriteLine($"privateKeyForTestNet: {privateKeyForTestNet}"); 
//Output:
//cVY5auviDh8LmYUW8AfafseD6p6uFoZrP7GjS3rzAerpRKE9Wmuz

//You can also generate a private key by GetWif() method on a privateKey by specifying a network type.
//Note that we're using the same privateKey generated from above.
bool WifIsBitcoinSecret = mainNetPrivateKey == privateKey.GetWif(Network.Main);
Console.WriteLine(WifIsBitcoinSecret); 
//Output:
//True
```  

Note that it is easy to go from **BitcoinSecret** to private **Key**. On the other hand, it is impossible to go from a Bitcoin Address to Public Key because the Bitcoin Address contains a hash of the Public Key, not the Public Key itself.  
Process this information by examining the similarities between these two codeblocks:  

```cs
Key privateKey = new Key(); // generate a random private key
BitcoinSecret bitcoinSecret = privateKey.GetWif(Network.Main); // L5B67zvrndS5c71EjkrTJZ99UaoVbMUAK58GKdQUfYCpAa6jypvn
Key samePrivateKey = bitcoinSecret.PrivateKey;
Console.WriteLine(samePrivateKey == privateKey); // True
```  

```cs
PubKey publicKey = privateKey.PubKey;
BitcoinPubKeyAddress bitcoinPublicKey = publicKey.GetAddress(Network.Main); // 1PUYsjwfNmX64wS368ZR5FMouTtUmvtmTY
//PubKey samePublicKey = bitcoinPublicKey.ItIsNotPossible;
```  

### Exercise:
1. Generate a private key on the mainnet and note it.
2. Get the corresponding address.
3. Send bitcoins to it. As much as you cannot afford to lose, so it will keep you focused and motivated to get them back during the following lessons.
