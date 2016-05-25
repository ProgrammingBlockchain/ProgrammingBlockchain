## P2SH (Pay To Script Hash) {#p2sh-pay-to-script-hash}

As seen previously, Multi-Sig works easily in code, however, before p2sh, there was no way to ask a customer to pay to a multi-sig ```scriptPubKey``` as easily as we could hand him a ```BitcoinAddress```.  

**P2SH**, or **Pay To Script Hash**, is an easy way to represent any ```scriptPubKey``` as a simple ```BitcoinScriptAddress```, no matter how complicated it is.

In the previous part we generated this multisig:

```cs
Key bob = new Key();
Key alice = new Key();
Key satoshi = new Key();

var scriptPubKey = PayToMultiSigTemplate
    .Instance
    .GenerateScriptPubKey(2, new[] { bob.PubKey, alice.PubKey, satoshi.PubKey });

Console.WriteLine(scriptPubKey);
```  

```
2 0282213c7172e9dff8a852b436a957c1f55aa1a947f2571585870bfb12c0c15d61 036e9f73ca6929dec6926d8e319506cc4370914cd13d300e83fd9c3dfca3970efb 0324b9185ec3db2f209b620657ce0e9a792472d89911e0ac3fc1e5b5fc2ca7683d 3 OP_CHECKMULTISIG
```  

Complicated isn’t it?

Instead, let’s see how such ```scriptPubKey``` would look like as a **P2SH** payment.

```cs
Key bob = new Key();
Key alice = new Key();
Key satoshi = new Key();

var paymentScript = PayToMultiSigTemplate
    .Instance
    .GenerateScriptPubKey(2, new[] { bob.PubKey, alice.PubKey, satoshi.PubKey }).PaymentScript;

Console.WriteLine(paymentScript);
```  

```
OP_HASH160 57b4162e00341af0ffc5d5fab468d738b3234190 OP_EQUAL
```  

Do you see the difference? This p2sh ```scriptPubKey``` represents the hash of my multi-sig script: ```redeemScript.Hash.ScriptPubKey```

Since it is a hash, you can easily convert is as a base58 string ```BitcoinScriptAddress```.

```cs
Key bob = new Key();
Key alice = new Key();
Key satoshi = new Key();

Script redeemScript =
    PayToMultiSigTemplate
    .Instance
    .GenerateScriptPubKey(2, new[] { bob.PubKey, alice.PubKey, satoshi.PubKey });
//Console.WriteLine(redeemScript.Hash.ScriptPubKey);
Console.WriteLine(redeemScript.Hash.GetAddress(Network.Main)); // 3E6RvwLNfkH6PyX3bqoVGKzrx2AqSJFhjo
```  

Such address is understood by any client wallet. Even if such wallet does not understand what “multi sig” is.

In P2SH payment, we refer as the **Redeem Script**, the ```scriptPubKey``` that got hashed.  

![](../assets/RedeemScript.png)  

Since the payer only knows about the **Hash of the RedeemScript**, he does not know the **Redeem Script**, and so, in our case, don’t even have to know that he is sending money to a multi sig of Bob/Satoshi/Alice.  

Signing such transaction is similar to what we have done before. The only difference is that you have to provide the **Redeem Script** when you build the Coin for the **TransactionBuilder**

Imagine that the multi sig P2SH receive a coin in a transaction called ```received```.  

```cs
Script redeemScript =
    PayToMultiSigTemplate
    .Instance
    .GenerateScriptPubKey(2, new[] { bob.PubKey, alice.PubKey, satoshi.PubKey });
////Console.WriteLine(redeemScript.Hash.ScriptPubKey);
//Console.WriteLine(redeemScript.Hash.GetAddress(Network.Main));
            
Transaction received = new Transaction();
//Pay to the script hash
received.Outputs.Add(new TxOut(Money.Coins(1.0m), redeemScript.Hash));
```  

> Warning: The payment is sent to ```redeemScript.Hash``` and not to ```redeemScript```!  

Then, once alice/bob/satoshi want to spend what they received, instead of creating a ```Coin``` they create a ```ScriptCoin```.  

```cs
//Give the redeemScript to the coin for Transaction construction
//and signing
ScriptCoin coin = received.Outputs.AsCoins().First()
                                    .ToScriptCoin(redeemScript);
```  

![](../assets/ScriptCoin.png)  

The rest of the code concerning transaction generation and signing is exactly the same as in the previous part with native multi sig.
