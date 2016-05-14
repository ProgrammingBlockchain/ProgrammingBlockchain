# Payment script {#payment-script}
You might not know that as far as the Blockchain is concerned, there is no such thing as a Bitcoin Address. Internally, the Bitcoin protocol identifies the recipient of Bitcoin by a payment script, we call it **ScriptPubKey**.  

![](../assets/ScriptPubKey.png)  
A ScriptPubKey may look like this:  
```OP_HASH160 14836dbe7f38c5ac3d49e8d790af808a4ee9edcf OP_EQUALVERIFY OP_CHECKSIG```  

It is a short script that explains what conditions must be met to claim ownership of bitcoins. We will go into the types of instructions that can be given in a ScriptPubKey as we move through the lessons of this book.  

We are able to generate the scriptPubKey from the Bitcoin Address. This is a step that all bitcoin clients do to translate the “human friendly” Bitcoin Address to the Blockchain readable address.

![](../assets/BitcoinAddressToScriptPubKey.png)  

```cs 
Console.WriteLine(mainNetAddress.ScriptPubKey); // OP_DUP OP_HASH160 14836dbe7f38c5ac3d49e8d790af808a4ee9edcf OP_EQUALVERIFY OP_CHECKSIG
Console.WriteLine(testNetAddress.ScriptPubKey); // OP_DUP OP_HASH160 14836dbe7f38c5ac3d49e8d790af808a4ee9edcf OP_EQUALVERIFY OP_CHECKSIG
Console.WriteLine(publicKeyHash); // 14836dbe7f38c5ac3d49e8d790af808a4ee9edcf
```  

Notice the ScriptPubKey for testnet and mainnet address is the same?  
Notice the ScriptPubKey contains the hash of the public key?  
We will not go into the details yet, but note that the ScriptPubKey appears to have nothing to do with the Bitcoin Address, but it does show the hash of the public key.  

Bitcoin Addresses are composed of a network identifier and the hash of a public key. Knowing this, it is possible to generate a bitcoin address from the scriptPubKey and the network identifier.

```cs
var paymentScript = publicKeyHash.ScriptPubKey;
var sameMainNetAddress = paymentScript.GetDestinationAddress(Network.Main);
Console.WriteLine(mainNetAddress == sameMainNetAddress); // True
```   

It is also possible to retrieve the hash from the payment script (ScriptPubKey) and generate a Bitcoin Address from it:  

```cs
var samePublicKeyHash = (KeyId) paymentScript.GetDestination();
Console.WriteLine(publicKeyHash == samePublicKeyHash); // True
var sameMainNetAddress2 = new BitcoinPubKeyAddress(samePublicKeyHash, Network.Main);
Console.WriteLine(mainNetAddress == sameMainNetAddress2); // True
```   

> **Note:** The payment script (ScriptPubKey) not necessarily contains the hashed public key(s) permitted to spend the bitcoin.  

So now you understand the relationship between a Private Key, a Public Key, a Public Key Hash, a Bitcoin Address and a ScriptPubKey. Let me sum it up with the whole diagram and the corresponding code:  








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
 






So now you understand the relationship between a Private Key, a Public Key, a Public Key Hash, a Bitcoin Address and a scriptPubKey.

Private keys are often represented in Base58Check called a **Bitcoin Secret** (also known as **Wallet Import Format** or simply **WIF**), like Bitcoin Addresses.

For the rest of the book you will use an address that you have generated for yourself.

Note that it is easy to go from Bitcoin Secret to Private Key. It is important to remember that it is impossible to go from a Bitcoin Address to Public Key because the Bitcoin Address contains a hash of the Public Key, not the Public Key itself.

Bitcoin Secret: KyVVPaNYFWgSCwkvhMG3TruG1rUQ5o7J3fX7k8w7EepQuUQACfwE

Copy Bitcoin Secret you are presented, and add the following code to your main method in Program.cs, substituting the secret you were given for the one we have entered.

**Exercise:** Note your own generated private key that you will use in the rest of this book along with its address. I will store my private key in the variable **BitcoinSecret** **paymentSecret** for the rest of this book.

**Exercise:** Get the **Bitcoin Address** of the **paymentSecret**, store in in **paymentAddress**, and send some money on it from **Bitcoin Core**. Send something like 0.01 BTC, you can increase when if you feel more confident. ;)

