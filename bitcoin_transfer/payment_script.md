## Chapter2. ScriptPubKey {#payment-script}
You might not know that, as far as the Blockchain is concerned, there is no such thing as a Bitcoin address. Internally, the Bitcoin protocol identifies the recipient of Bitcoin by a **ScriptPubKey**.  

![](../assets/ScriptPubKey.png)  
A **ScriptPubKey** may look like this:  
```OP_DUP OP_HASH160 14836dbe7f38c5ac3d49e8d790af808a4ee9edcf OP_EQUALVERIFY OP_CHECKSIG```  

It is a short script that explains what conditions must be met to claim ownership of bitcoins. We will go into the types of operations in a **ScriptPubKey** as we move through the lessons of this book.  

We are able to generate the ScriptPubKey from the Bitcoin Address. This is a step that all bitcoin clients do to translate the “human friendly” Bitcoin Address to the Blockchain readable address.

![](../assets/BitcoinAddressToScriptPubKey.png)  

```cs 
var publicKeyHashByHardCodedValue = new KeyId("97eb9da945f16139acb552a2de4081eb7f5176d9");
Console.WriteLine($"publicKeyHashBySimpleWay: {publicKeyHashByHardCodedValue}");
//Output:
//97eb9da945f16139acb552a2de4081eb7f5176d9

//Generate bitcoin addresses for each network.
var bitcoinAddressForTestNet = publicKeyHashByHardCodedValue.GetAddress(Network.TestNet);
var bitcoinAddressForMainNet = publicKeyHashByHardCodedValue.GetAddress(Network.Main);
Console.WriteLine($"bitcoinAddressForTestNet {bitcoinAddressForTestNet}");
//Output:
//muNEau1yEUXmTcU8ns6GYhHfSCsMjseuNB
Console.WriteLine($"bitcoinAddressForMainNet {bitcoinAddressForMainNet}");
//Output:
//1ErHHqvzRT6WgVzX5J7tin5LaDGev2UobP

//Generate a ScriptPubKey from a Bitcoin address.
Console.WriteLine(bitcoinAddressForMainNet.ScriptPubKey);
//Output:
//OP_DUP OP_HASH160 97eb9da945f16139acb552a2de4081eb7f5176d9 OP_EQUALVERIFY OP_CHECKSIG
Console.WriteLine(bitcoinAddressForTestNet.ScriptPubKey);
//Output:
//OP_DUP OP_HASH160 97eb9da945f16139acb552a2de4081eb7f5176d9 OP_EQUALVERIFY OP_CHECKSIG
```  

Can you notice the ScriptPubKeys (bitcoinAddressForMainNet.ScriptPubKey and bitcoinAddressForTestNet.ScriptPubKey) generated from each Bitcoin address for the TestNet and the MainNet are exactly identical?  
Can you notice the **ScriptPubKey** contains the hash of the public key hash (97eb9da945f16139acb552a2de4081eb7f5176d9)?  
We will not go into the details yet. However, the **ScriptPubKey** is looking like it has nothing to do with the Bitcoin address, just with showing the public key hash value from a part of the entire ScriptPubKey.  

Bitcoin addresse is composed of a "version byte" which identifies the network type where for the bitcoin address to being used and the "hash of a public key". So we can go backwards and generate a bitcoin address from the **ScriptPubKey** and the network identifier.

```cs
//Generate a ScriptPubKey from publicKeyHash.
var scriptPubKeyForPayment = publicKeyHashForThisExample.ScriptPubKey;

//Get a Bitcoin address by specifying a network identifier on the ScriptPubKey.
var bitcoinAddressFromSPKForMainNet = scriptPubKeyForPayment.GetDestinationAddress(Network.Main);
Console.WriteLine(bitcoinAddressForMainNet == bitcoinAddressFromSPKForMainNet);
//Output:
//True
```   

It is also possible to retrieve the public key hash value from the ScriptPubKey and also generate a Bitcoin address by specifying a network identifier on the public key hash generated in this way.  

```cs
//Get the public key hash value from the ScriptPubKey.
var publicKeyHashFromSPK = (KeyId)scriptPubKeyForPayment.GetDestination();
Console.WriteLine(publicKeyHashForThisExample == publicKeyHashFromSPK); 
//Output:
//True

//Get the Bitcoin address by using the publick key hash value retrieved from the ScriptPubKey from above code and network identifier.
var bitcoinAddressGotAgain = new BitcoinPubKeyAddress(publicKeyHashFromSPK, Network.Main);
Console.WriteLine(bitcoinAddressForMainNet == bitcoinAddressGotAgain);
//Output:
//True
```   

> **Note:** A ScriptPubKey does not necessarily contain the hashed public key(s) permitted to spend the bitcoin like this:  
```
OP_DUP OP_HASH160 <Omitted public key hash value> OP_EQUALVERIFY OP_CHECKSIG
```

So now you understand the relationship between a Private Key, a Public Key, a Public Key Hash, a Bitcoin Address and a ScriptPubKey.

In the remainder of this book, we will exclusively use a **ScriptPubKey**.  
And again, a Bitcoin address representing a recipient is just the address human-readable for the UI layer like a wallet software. The address representing a recipient, which is blockchain system readable, is the ScriptPubKey which is used in a TxOut of a transaction.
