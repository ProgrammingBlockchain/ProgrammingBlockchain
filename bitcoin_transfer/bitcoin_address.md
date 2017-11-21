# Chapter1. Bitcoin address {#bitcoin-address}

## Section1. Bitcoin address

You know that your **Bitcoin address** is what you share to the world to get paid.  
![](../assets/BitcoinAddress.png)  
You probably know that your wallet software uses a **private key** to spend the money you received on this address.  
![](../assets/PrivateKey.png)  

The keys are not stored on the network and they can be generated without access to the Internet.  

This is how you generate a private key with NBitcoin:  
```cs  
//Generate a random private key.
Key privateKey = new Key(); 
Console.WriteLine($"privateKey: {privateKey.ToString(Network.Main)}");
//Output:
//L3gyRGQ8Da1mMnDFkM9sFYbTV7P8hN8vHzmkALsfkyK7Wjfby5ZB
```  
Note that above code is showing a generated private key by internally instantiating BitcoinSecret class with passing this private key object containing the private key data into the BitcoinSecret constructor as the first argument. And subsequently BitcoinSecret instantiates its base class, Base58Data, with passing received the key object and the network identifier.



From a private key, we use a one-way cryptographic function, to generate a **public key**. 

![](../assets/PrivKeyPubKey.png)  
```cs 
PubKey publicKey = privateKey.PubKey;
Console.WriteLine(publicKey); 
//Output:
//0251036303164f6c458e9f7abecb4e55e5ce9ec2b2f1d06d633c9653a07976560c
```

## Section2. Dive into more details of "new Key()", a private key, a public key.

You're probably doubtful when you see a "Key privateKey = new Key()" from above code, saying "Should the variable name be named by keyObject rather than a privateKey?

Yes, it is a key object. If you print a privateKey variable by:
```cs
Console.WriteLine(privateKey);
```
, you'll see the output like this:
```cs
NBitcoin.Key
```

Moreover, you can do lots of tasks by this key object. Try to examine what you can do by this key object by using IntelliSense like this:
```cs
Console.WriteLine(privateKey.);
```

You're going to be sure it's a key object once you examine a Key class, because a Key class looks like an ordinary class including lots of its members.

It's true that an object which is created by "new Key()" is a key object.

**However, in the NBitcoin, we often use an object which is created by "new Key()" as a private key.**

When you instantiate a key object, under the hood, in the case of the NBitcoin which is run on the .NET environment, you invoke by default a secure RNG to generate a random key data and store it into a key object.  

For more details, reference a "Is it random enough?" chapter of "Key generation and encryption" part.

As mentioned above, a key object generated in this way contains a randomly generated key data in the object, and this object is often used in generating other types of keys such as a public key, BitcoinSecret(nothing but a private key represented in Base58Check to use it in the UI layer such as a wallet software, and Bitcoin secret is exactly identical concept with a WIF except for its differently called name.), a ScriptPubKey and so on.


## Section3. Bitcoin network and Bitcoin address
There are two Bitcoin **networks**: 
* **TestNet** is a Bitcoin network for development purposes. Bitcoins on this network worth nothing.  
* **MainNet** is the Bitcoin network everybody uses.  

> **Note:** You can acquire testnet coins quickly by using **faucets**. Just google "get testnet bitcoins".  

You can easily get your **Bitcoin address** from your public key and the **network** on which this address should be used. 

![](../assets/PubKeyToAddr.png)  

```cs 
Console.WriteLine(publicKey.GetAddress(Network.Main)); 
//Output:
//1PUYsjwfNmX64wS368ZR5FMouTtUmvtmTY
Console.WriteLine(publicKey.GetAddress(Network.TestNet)); 
//Output:
//n3zWAo2eBnxLr3ueohXnuAa8mTVBhxmPhq
```  
Note that a Bitcoin address for mainnet starts with "1", and a Bitcoin address for testnet starts with "m" or "n".

**To be precise, a Bitcoin address is made up of a "version byte" which is different on both networks(mainnet, testnet). But this version byte identifies the network type. And a Bitcoin address is also made of your "public key’s hash bytes". Both of these bytes are concatenated and then encoded into a Base58Check encoding scheme which has an additional 4 bytes checksum data, compared to a Base58:**  

In other words, it means that a generated Bitcoin address is always in Base58Check encoding scheme.(If this is wrong, please edit it.)

![](../assets/PubKeyHashToBitcoinAddress.png)  

The above illustration shows a standard way of generating a Bitcoin address by processing entire steps(Public key -> Public key hash + Network => Bitcoin address), with not using a sugar syntax for generating a Bitcoin address(Public key + Network => Bitcoin address).

```cs 
var publicKeyHash = publicKey.Hash;
Console.WriteLine(publicKeyHash);
//Output:
//f6889b21b5540353a29ed18c45ea0031280c42cf

//Get a Bitcoin address for a mainnet by publicKeyHash and network identifier.
var mainNetAddress = publicKeyHash.GetAddress(Network.Main);
var testNetAddress = publicKeyHash.GetAddress(Network.TestNet);

Console.WriteLine(mainNetAddress); 
//Output:
//1PUYsjwfNmX64wS368ZR5FMouTtUmvtmTY
Console.WriteLine(testNetAddress); 
//Output:
//n3zWAo2eBnxLr3ueohXnuAa8mTVBhxmPhq
```  

> **Fact:** Internally, a **public key hash** is generated by using a SHA256 cryptographic hash function on the public key, then a RIPEMD160 cryptographic hash function on the result, using Big Endian notation. The function could look like this: RIPEMD160(SHA256(publickey))  

## Section4. Encoding schemes
There is a one binary-to-text encoding scheme which is called Base64.
Base64 represents binary data in an ASCII string format with 64 characters composed of A-Z, a-z, 0-9, and +(plus) ,/(slash).

Base58 is a modification of Base64 which was first suggested by Satoshi Nakamoto for Bitcoin system implementation.
Base58 uses 58 characters by eleminating 6 characters from Base64 such as 0(zero), O(uppercase o), I(uppercase i), l(lowercase L), +(plus), and /(slash) to avoid a mistake by an ambiguity of characters.

The Base58Check, a modification of Base58, encoding scheme has some neat features, such as checksums to prevent typos aside from a lack of ambiguous characters such as '0' and 'O' from Base58.  
The Base58Check encoding scheme also provides a consistent way to determine the network of a given address, which means that this feature prevents a wallet from sending MainNet coins to a TestNet address.

```cs
var bitcoinAddressForMainNet = keyObj.PubKey.GetAddress(Network.Main);
var bitcoinAddressForTestNet = keyObj.PubKey.GetAddress(Network.TestNet);

//Get the consistent network type from the specific Bitcoin address.
var mainNetFromBitcoinAddress = bitcoinAddressForMainNet.Network;
var testNetFromBitcoinAddress = bitcoinAddressForTestNet.Network;
```

> **Tip:** Practicing a Bitcoin Programming on the MainNet makes mistakes more memorable.  
