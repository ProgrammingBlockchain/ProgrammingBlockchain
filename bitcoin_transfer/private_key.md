# Private Key {#private-key}

Private keys are often represented in Base58Check called a **Bitcoin Secret** (also known as **Wallet Import Format** or simply **WIF**), like Bitcoin Addresses.

Note that it is easy to go from Bitcoin Secret to Private Key. It is important to remember that it is impossible to go from a Bitcoin Address to Public Key because the Bitcoin Address contains a hash of the Public Key, not the Public Key itself.

Bitcoin Secret: KyVVPaNYFWgSCwkvhMG3TruG1rUQ5o7J3fX7k8w7EepQuUQACfwE

Copy Bitcoin Secret you are presented, and add the following code to your main method in Program.cs, substituting the secret you were given for the one we have entered.

**Exercise:** Note your own generated private key that you will use in the rest of this book along with its address. I will store my private key in the variable **BitcoinSecret** **paymentSecret** for the rest of this book.

**Exercise:** Get the **Bitcoin Address** of the **paymentSecret**, store in in **paymentAddress**, and send some money on it from **Bitcoin Core**. Send something like 0.01 BTC, you can increase when if you feel more confident. ;)









```cs  
Key privateKey = new Key(); // generate a random private key
var mainNetPrivateKey = privateKey.GetBitcoinSecret(Network.Main);  // get our private key for the mainnet
var testNetPrivateKey = privateKey.GetBitcoinSecret(Network.TestNet);  // get our private key for the testnet
Console.WriteLine(mainNetPrivateKey); // L5B67zvrndS5c71EjkrTJZ99UaoVbMUAK58GKdQUfYCpAa6jypvn
Console.WriteLine(testNetPrivateKey); // cVY5auviDh8LmYUW8AfafseD6p6uFoZrP7GjS3rzAerpRKE9Wmuz
```  
```cs 
PubKey publicKey = privateKey.PubKey;
Console.WriteLine(publicKey); // 0251036303164f6c458e9f7abecb4e55e5ce9ec2b2f1d06d633c9653a07976560c
bool isPublicKeyNetworkIndependent =
    (publicKey == mainNetPrivateKey.PubKey)
    &&
    (publicKey == testNetPrivateKey.PubKey);
Console.WriteLine(isPublicKeyNetworkIndependent); // True
```  
 





