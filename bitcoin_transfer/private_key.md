## Private Key {#private-key}

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

Note that it is easy to go from Bitcoin Secret to Private Key. It is important to remember that it is impossible to go from a Bitcoin Address to Public Key because the Bitcoin Address contains a hash of the Public Key, not the Public Key itself.  
Process this information by examining the similarities between this two codeblock:  

```cs
BitcoinSecret bitcoinPrivateKey = privateKey.GetWif(Network.Main); // L5B67zvrndS5c71EjkrTJZ99UaoVbMUAK58GKdQUfYCpAa6jypvn
Key samePrivateKey = bitcoinPrivateKey.PrivateKey;
Console.WriteLine(privateKey == samePrivateKey); // True
```  

```cs
PubKey publicKey = privateKey.PubKey;
BitcoinPubKeyAddress bitcoinPubicKey = publicKey.GetAddress(Network.Main); //
//PubKey samePublicKey = bitcoinPubicKey.ItIsNotPossible;
```  

### Exercise:
1. Generate a private key on the mainnet and note it.
2. Get the corresponding address and send some bitcoins to it..  

Send something like 1-10 USD worth of coins, you can increase when if you feel more confident;)  

Working on the mainnet will make sure you are going to finish the next chapter. If you are less motivated don't possess any bitcoin just use the testnet. You can acquire testnet coins quickly, just google "get testnet bitcoins", they are also called **faucets**.  


 





