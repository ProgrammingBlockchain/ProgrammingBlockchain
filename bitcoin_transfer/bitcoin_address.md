## Bitcoin Address {#bitcoin-address}

You know that your **Bitcoin Address** is what you share to the world to get paid.  
![](../assets/BitcoinAddress.png)  
You probably know that your wallet software uses a **private key** to spend the money you received on this address.  
![](../assets/PrivateKey.png)  

The keys are not stored in the network and they can be generated without access to the Internet.  

With code:  
```cs  
Key privateKey = new Key(); // generate a random private key
```  
> **([Mastering Bitcoin](https://github.com/bitcoinbook/bitcoinbook/))** The size of bitcoin’s private key space, 2256 is an unfathomably large number. It is approximately 1077 in decimal. For comparison, the visible universe is estimated to contain 1080 atoms.  
Keys come in pairs consisting of a private (secret) key and a public key. Think of the public key as similar to a bank account number and the private key as similar to the secret PIN, or signature on a check that provides control over the account. These digital keys are very rarely seen by the users of bitcoin. For the most part, they are stored inside the wallet file and managed by the bitcoin wallet software.  
From the private key, we use elliptic curve multiplication, a one-way cryptographic function, to generate a **public key**.  

![](../assets/PrivKeyPubKey.png)  
```cs 
PubKey publicKey = privateKey.PubKey;
Console.WriteLine(publicKey); // 0251036303164f6c458e9f7abecb4e55e5ce9ec2b2f1d06d633c9653a07976560c
```  

> **Fact:** TestNet is a Bitcoin network for development purposes. Bitcoins on this network worth nothing.  MainNet is the bitcoin network everybody knows.  

You can magically get your **bitcoin address** from your public key and by specifying the **network**. *(You should enjoy this magic now, because I will clear it up shortly.)*  

![](../assets/PubKeyToAddr.png)  

```cs 
Console.WriteLine(publicKey.GetAddress(Network.Main)); // 1PUYsjwfNmX64wS368ZR5FMouTtUmvtmTY
Console.WriteLine(publicKey.GetAddress(Network.TestNet)); // n3zWAo2eBnxLr3ueohXnuAa8mTVBhxmPhq
```  

**More accurately a bitcoin address is made up of a Base58check encoded combination of your public key’s hash and some information about the network the address is for:**  

```cs 
var publicKeyHash = publicKey.Hash;
Console.WriteLine(publicKeyHash); // f6889b21b5540353a29ed18c45ea0031280c42cf
BitcoinAddress mainNetAddress = publicKeyHash.GetAddress(Network.Main);
BitcoinAddress testNetAddress = publicKeyHash.GetAddress(Network.TestNet);
Console.WriteLine(mainNetAddress); // 1PUYsjwfNmX64wS368ZR5FMouTtUmvtmTY
Console.WriteLine(testNetAddress); // n3zWAo2eBnxLr3ueohXnuAa8mTVBhxmPhq
```  




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
 
The Base58Check encoding has some neat features, such as checksums to prevent typos and a lack of ambiguous characters such as “0” and “O.”

Fact: **TestNet** is a bitcoin network for development purposes, the bitcoin on this network are worth nothing. **MainNet** is the bitcoin network everybody knows.

You might not know that as far as the Blockchain is concerned, there is no such thing as a Bitcoin Address. Internally, the Bitcoin protocol identifies the recipient of Bitcoin by a **ScriptPubKey**. A ScriptPubKey is a short script that explains what conditions must be met to claim ownership of bitcoins. We will go into the types of instructions that can be given in a ScriptPubKey as we move through the lessons of this book. The ScriptPubKey may contain the hashed public key(s) permitted to spend the bitcoin.

Fact: Practicing Bitcoin Programming on MainNet makes mistakes more memorable.

This diagram illustrates the relationships between the public key, private key, bitcoin address, and the ScriptPubKey.

Now we can show you the relationship in code. Open Chapter1.cs, add “using NBitcoin;” to the top and then make the following method:

Public Key: 02fc2e5ab52c89f3bfbbbd702254ab9fc7134173e5682b69bf740c3d0bf6bfc4ce

Hashed public key: 1b2da6ee52ac5cd5e96d2964f12a0241851f8d2a

Address: 13Uhw9BmdaXbnjDXiEd4HU4yesj7kKjxCo

ScriptPubKey from address: OP_DUP OP_HASH160 1b2da6ee52ac5cd5e96d2964f12a0241851f8d2a OP_EQUALVERIFY OP_CHECKSIG

ScriptPubKey from hash: OP_DUP OP_HASH160 1b2da6ee52ac5cd5e96d2964f12a0241851f8d2a OP_EQUALVERIFY OP_CHECKSIG

Press F5 and examine the output. You just learned how to create a private key, the corresponding public key, the public key’s hash, the address, and the scriptPubKey.

We will not go into the details yet, but note that the ScriptPubKey appears to have nothing to do with the BitcoinAddress, but it does show the hash of the public key. Notice how we were able to generate the scriptPubKey from the Bitcoin Address? This is a step that all bitcoin clients do to translate the “human friendly” Bitcoin Address to the Blockchain readable address.

Bitcoin Addresses are composed of a network identifier and the hash of a public key. Knowing this, it is possible to generate a bitcoin address from the scriptPubKey and the network identifier as the following code demonstrates.

Bitcoin Address: 13Uhw9BmdaXbnjDXiEd4HU4yesj7kKjxCo

It is also possible to retrieve the hash from the scriptPubKey and generate a Bitcoin Address from it as we showed in Lesson1().

Public Key Hash: 1b2da6ee52ac5cd5e96d2964f12a0241851f8d2a

Bitcoin Address: 13Uhw9BmdaXbnjDXiEd4HU4yesj7kKjxCo

Fact: The hash of the public key is generated by performing a SHA256 hash on the public key, and then performing a RIPEMD160 hash on the result, with Big Endian notation. The function could look like this: RIPEMD160(SHA256(pubkey))

So now you understand the relationship between a Private Key, a Public Key, a Public Key Hash, a Bitcoin Address and a scriptPubKey.

Private keys are often represented in Base58Check called a **Bitcoin Secret** (also known as **Wallet Import Format** or simply **WIF**), like Bitcoin Addresses.

For the rest of the book you will use an address that you have generated for yourself.

Note that it is easy to go from Bitcoin Secret to Private Key. It is important to remember that it is impossible to go from a Bitcoin Address to Public Key because the Bitcoin Address contains a hash of the Public Key, not the Public Key itself.

Bitcoin Secret: KyVVPaNYFWgSCwkvhMG3TruG1rUQ5o7J3fX7k8w7EepQuUQACfwE

Copy Bitcoin Secret you are presented, and add the following code to your main method in Program.cs, substituting the secret you were given for the one we have entered.

**Exercise:** Note your own generated private key that you will use in the rest of this book along with its address. I will store my private key in the variable **BitcoinSecret** **paymentSecret** for the rest of this book.

**Exercise:** Get the **Bitcoin Address** of the **paymentSecret**, store in in **paymentAddress**, and send some money on it from **Bitcoin Core**. Send something like 0.01 BTC, you can increase when if you feel more confident. ;)