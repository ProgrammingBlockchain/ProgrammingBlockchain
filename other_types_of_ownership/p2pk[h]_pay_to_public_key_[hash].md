## P2PK\[H\] \(Pay to Public Key \[Hash\]\) {#p2pk-h-pay-to-public-key-hash}

### P2PKH - Quick recap

Kita telah mempelajari bahwa **Address** **Bitcoin **adalah **hash **dari sebuah **public key**:

```cs
var publicKeyHash = new Key().PubKey.Hash;
var bitcoinAddress = publicKeyHash.GetAddress(Network.Main);
Console.WriteLine(publicKeyHash); // 41e0d7ab8af1ba5452b824116a31357dc931cf28
Console.WriteLine(bitcoinAddress); // 171LGoEKyVzgQstGwnTHVh3TFTgo5PsqiY
```

Kita juga mempelajari beberapa hal mendasar pada blockchain, dan mengetahui juga bahwa tidak ada yang namanya **address bitcoin** sebenarnya. Karena blockchain mengidentifikasi penerima \(receiver\) dengan sebuah **ScriptPubKey**, dan **ScriptPubKey,** yang memungkinkan untuk generate key dari address:

```cs
var scriptPubKey = bitcoinAddress.ScriptPubKey;
Console.WriteLine(scriptPubKey); // OP_DUP OP_HASH160 41e0d7ab8af1ba5452b824116a31357dc931cf28 OP_EQUALVERIFY OP_CHECKSIG
```

And vice versa:

```cs
var sameBitcoinAddress = scriptPubKey.GetDestinationAddress(Network.Main);
```

### P2PK

However, all **ScriptPubKey** does not represent a Bitcoin Address. For example the first transaction in the blockchain, called the genesis:

```cs
Block genesisBlock = Network.Main.GetGenesis();
Transaction firstTransactionEver = genesisBlock.Transactions.First();
var firstOutputEver = firstTransactionEver.Outputs.First();
var firstScriptPubKeyEver = firstOutputEver.ScriptPubKey;
var firstBitcoinAddressEver = firstScriptPubKeyEver.GetDestinationAddress(Network.Main);
Console.WriteLine(firstBitcoinAddressEver == null); // True
```

```cs
Console.WriteLine(firstTransactionEver);
```

```json
{
…
  "out": [
    {
      "value": "50.00000000",
      "scriptPubKey": "04678afdb0fe5548271967f1a67130b7105cd6a828e03909a67962e0ea1f61deb649f6bc3f4cef38c4f35504e51ec112de5c384df7ba0b8d578a4c702b6bf11d5f OP_CHECKSIG"
    }
  ]
}
```

You can see the form of the **scriptPubKey** is different:

```cs
Console.WriteLine(firstScriptPubKeyEver); // 04678afdb0fe5548271967f1a67130b7105cd6a828e03909a67962e0ea1f61deb649f6bc3f4cef38c4f35504e51ec112de5c384df7ba0b8d578a4c702b6bf11d5f OP_CHECKSIG
```

A bitcoin address is represented by: **OP\_DUP &lt;hash&gt; OP\_EQUALVERIFY OP\_CHECKSIG** But here we have : **&lt;pubkey&gt; OP\_CHECKSIG**

In fact, at the beginning, **public key** were used directly in the **ScriptPubKey**.

```cs
var firstPubKeyEver = firstScriptPubKeyEver.GetDestinationPublicKeys().First();
Console.WriteLine(firstPubKeyEver); // 04678afdb0fe5548271967f1a67130b7105cd6a828e03909a67962e0ea1f61deb649f6bc3f4cef38c4f35504e51ec112de5c384df7ba0b8d578a4c702b6bf11d5f
```

Now we are mainly using the hash of the public key.

![](../assets/PPKH.png)

```cs
key = new Key();
Console.WriteLine("Pay to public key : " + key.PubKey.ScriptPubKey);
Console.WriteLine();
Console.WriteLine("Pay to public key hash : " + key.PubKey.Hash.ScriptPubKey);
```

```
Pay to public key : 02fb8021bc7dedcc2f89a67e75cee81fedb8e41d6bfa6769362132544dfdf072d4 OP_CHECKSIG
Pay to public key hash : OP_DUP OP_HASH160 0ae54d4cec828b722d8727cb70f4a6b0a88207b2 OP_EQUALVERIFY OP_CHECKSIG
```

These 2 types of payment are referred as **P2PK** \(pay to public key\) and **P2PKH** \(pay to public key hash\).

Satoshi later decided to use P2PKH instead of P2PK for two reasons:

* Elliptic Curve Cryptography, the cryptography used by your **public key** and **private key**\) is vulnerable to a modified Shor's algorithm for solving the discrete logarithm problem on elliptic curves. In plain English, it means that, with a quantum computer, in theory, it is possible in some distant future to **retrieve a private key from a public key**. By publishing the public key only when the coin are spend, such attack is rendered ineffective. \(Assuming addresses are not reused.\) 
* The hash being smaller \(20 bytes\), it is smaller to print, and easier to embed into small storage like a QR code.

Nowadays, there is no reason to use P2PK directly, but it is still used in combination with P2SH, more on this later.

> \([Discussion](https://www.reddit.com/r/Bitcoin/comments/4isxjr/petition_to_protect_satoshis_coins/d30we6f)\) If the early use of P2PK will not be addressed, it will have a serious impact on the Bitcoin price.

### Exercise

\([nopara73](https://github.com/nopara73)\) While reading this chapter I found the the abbreviations \(P2PK, P2PKH, P2W, etc..\) very confusing.  
My trick was to force myself to pronounce the terms fully every time I encountered them during the following lessons. Suddenly everything made much more sense. I recommend you to do the same.

