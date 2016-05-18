## P2PK[H] (Pay to Public Key [Hash]) {#p2pk-h-pay-to-public-key-hash}

In part 2 we learned that a **Bitcoin Address** was the **hash of a** **public key**.We also learned that as far as the blockchain is concerned, there is no such thing as a **bitcoin address**. The blockchain identifies a receiver with a **ScriptPubKey**, and such **ScriptPubKey** could be generated from the address. (and vice versa)  

```cs
Key key = new Key();
BitcoinAddress address = key.PubKey.GetAddress(Network.Main);
Console.WriteLine(address.ScriptPubKey);
```  

```
OP_DUP OP_HASH160 c86bf818bfd2b4943c003464815a8259c0de5e59 OP_EQUALVERIFY OP_CHECKSIG
```  

However, all **ScriptPubKey** does not represent a Bitcoin Address.

Here is an example of the first transaction in the blockchain. (The genesis)  

```cs
Console.WriteLine(Network.Main.GetGenesis().Transactions[0].ToString());
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

You can see the form of the **scriptPubKey** is different: A bitcoin address is represented by: **OP_DUP &lt;hash&gt; OP_EQUALVERIFY OP_CHECKSIG** But here we have : **&lt;pubkey&gt; OP_CHECKSIG**

In fact, at the beginning, **public key** were used directly in the **ScriptPubKey**.  
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

These 2 types of payment are referred as **P2PK** (pay to public key) and **P2PKH** (pay to public key hash).

Satoshi later decided to use P2PKH instead of P2PK for two reasons:

*   Elliptic Curve Cryptography, the cryptography used by your **public key** and **private key**) is vulnerable to a modified Shor's algorithm for solving the discrete logarithm problem on elliptic curves. In plain English, it means that, with a quantum computer, in theory, it is possible in some distant future to **retrieve a private key from a public key**.By publishing the public key only when the coin are spend, such attack is rendered ineffective. (assuming addresses are not reused)
*   The hash being smaller (20 bytes), it is smaller to print, and easier to embed into small storage like a QR code.

Nowadays, there is no reason to use P2PK directly, but it is still used in combination with P2SH, more on this later.