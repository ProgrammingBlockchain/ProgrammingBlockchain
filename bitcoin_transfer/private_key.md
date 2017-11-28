## Chapter3. Private key {#private-key}

Private keys are often represented in Base58Check encoding scheme, to use the encoded private key in the UI layer. And private key represented in Base58Check is especially called a **Bitcoin Secret**. Bitcoin secret is also known as a **Wallet Import Format** or simply **WIF** because a "Bitcoin secret" is usually used with a Bitcoin wallet which is a kind of an UI tool, along with a "Bitcoin address". Usually a Bitcoin secret is used for signing a signature for the coin and proof of ownership for the coin. And a Bitcoin address in the UI layer is used for representing a recipient to which the coin will be sent.   

![](../assets/BitcoinSecret.png)  

```cs  
//Generate one random private key.
Key privateKey = new Key();  

//Generate a Bitcoin secret for the MainNet, which is nothing but a private key represented in Base58Check binary-to-text encoding scheme.
BitcoinSecret privateKeyForMainNet = privateKey.GetBitcoinSecret(Network.Main);
//Generate a Bitcoin secret for the TestNet, which is nothing but a private key represented in Base58Check binary-to-text encoding scheme.
BitcoinSecret privateKeyForTestNet = privateKey.GetBitcoinSecret(Network.TestNet);

Console.WriteLine($"privateKeyForMainNet: {privateKeyForMainNet}");
//Output:
//L5DZpEdbDDhhk3EqtktmGXKv3L9GxttYTecxDhM5huLd82qd9uvo
Console.WriteLine($"privateKeyForTestNet: {privateKeyForTestNet}");
//Output:
//cVaZH9dSeHPxuUi7HAhtdqpyfZSgdLzEXgmRL7obD1zdNmxcW9aL

//You can also generate a private key by invoking GetWif() on the private key by additionally specifying a network identifier.  
//Note that we're using the same private key generated from above.  
BitcoinSecret privateKeyByGetWifMethod = privateKey.GetWif(Network.Main);
Console.WriteLine(privateKeyByGetWifMethod);
//Output:
//L5B67zvrndS5c71EjkrTJZ99UaoVbMUAK58GKdQUfYCpAa6jypvn

bool wifIsPrivateKey = privateKeyForMainNet == privateKey.GetWif(Network.Main);
Console.WriteLine(wifIsPrivateKey);
//Output:
//True
```  

Note that it is easy to go from a **Bitcoin secret** to a **private key**.
Recall that a Bitcoin secret is nothing but a private key just represented in Base58Check from a private key, which is often used in UI layer in Bitcoin system such as via wallet software. In the NBitcoin, when you instantiate a key object by a "new Key()", under the hood, you also invoke a secure PRNG to generate a random key data for a private key which will be stored in the key object.
On the other hand, it is impossible to go from a Bitcoin address to a public key because the Bitcoin address is generated from a hash of the public key(and + network identifier), not the public key itself.  

Process this information by examining the similarities between these two codeblocks:  

```cs
//Get the Bitcoin secret by invoking GetWif() on the private key with passing additionally the network identifier.
BitcoinSecret bitcoinSecretByGetWif = privateKey.GetWif(Network.Main);
//Get the Bitcoin secret by invoking GetBitcoinSecret() on the private key with passing additionally the network identifier.
BitcoinSecret bitcoinSecretByGetBitcoinSecret = privateKey.GetBitcoinSecret(Network.Main);

//Get the private key from the Bitcoin secret.
var privateKeyFromBsByGetWif = bitcoinSecretByGetWif.PrivateKey;
var privateKeyFromBsByGetBitcoinSecret = bitcoinSecretByGetBitcoinSecret.PrivateKey;

Console.WriteLine($"privateKeyFromBsByGetWif: {privateKeyFromBsByGetWif.ToString(Network.Main)}");
Console.WriteLine($"privateKeyFromBsByGetBitcoinSecret: {privateKeyFromBsByGetBitcoinSecret.ToString(Network.Main)}");
//L5DZpEdbDDhhk3EqtktmGXKv3L9GxttYTecxDhM5huLd82qd9uvo
//L5DZpEdbDDhhk3EqtktmGXKv3L9GxttYTecxDhM5huLd82qd9uvo
Console.WriteLine(privateKey==privateKeyFromBsByGetWif);
//Output:
//True
```  

You can get the Bitcoin address from the public key by additionally specifying the network identifier.
But it's impossible to get the public key from the Bitcoin address because the Bitcoin address is generated from the public key hash value(plus network identifier), not public key itself.

```cs
PubKey publicKey = privateKey.PubKey;
BitcoinPubKeyAddress bitcoinAddress = publicKey.GetAddress(Network.Main);
Console.WriteLine($"bitcoinAddress: {bitcoinAddress}");
//Output:
//1ErHHqvzRT6WgVzX5J7tin5LaDGev2UobP

//PubKey publicKeyFromBitcoinAddress = bitcoinAddress.ItIsNotPossible;
```  

### Exercise:
1. Generate a private key on the MainNet and note it.
2. Get the Bitcoin address related to that private key.
3. Send bitcoins to that Bitcoin address. As much as you cannot afford to lose, so it will keep you focused and motivated to get them back during the following lessons.
