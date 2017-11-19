## Arbitrary {#arbitrary}

From Bitcoin 0.10, the **RedeemScript** can be arbitrary, which means that with the script language of Bitcoin, you can create your own definition of what “ownership” means.

For example, in this scheme, imagine someone sends the coins to the Bitcoin address of this book, **1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB**, with adding an arbitrary redeem ScriptPubKey containing custum scripts which partially contain the hash value of my birth date. And whoever either knows "my date of birth" in the format of "dd/mm/yyyy", serialized in UTF-8 or the "private key" related to the Bitcoin address of this book can spend the coins by prooving the ownership for that one.

The details of the script language are out of scope of this book. You can easily find the documentation for the Bitcoin script language on various websites. And it is a stack based language so everyone having done with some assembler should be able to read it.  

> **Note:** ([nopara73](https://github.com/nopara73)) I find [Davide De Rosa's tutorial](http://davidederosa.com/basic-blockchain-programming/bitcoin-script-language-part-one/) as the most enjoyable one.

So first, let’s build the **RedeemScript**,  

> **Note:** For this code to work right, Click **References** -> **Add Reference** -> Find **System.Numerics**

```cs
BitcoinAddress bitcoinAddressOfThisBook = BitcoinAddress.Create("1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB");
var birthDate = Encoding.UTF8.GetBytes("18/07/1988");
var birthDateHash = Hashes.Hash256(birthDate);
Script redeemScript = new Script(
                "OP_IF "
                    + "OP_HASH256 " + Op.GetPushOp(birthDateHash.ToBytes()) + " OP_EQUAL " +
                "OP_ELSE "
                    + address.ScriptPubKey + " " +
                "OP_ENDIF");
```

This **RedeemScript** means that there are 2 ways of spending such **ScriptCoin**:  
Can be spent by anyone in either case he knows the data(birthDate) that can generate birthDateHash, or knows the private key related to the Bitcoin address of this book.  

Let’s say I sent money with such **redeemScriptPubKeyForSendingCoinToBook**:

```cs
var txForSendingCoinToBook = new Transaction();
txForSendingCoinToBook.Outputs.Add(new TxOut(Money.Parse("0.0001"), redeemScriptPubKeyForSendingCoinToBook.Hash));
ScriptCoin scriptCoinForSendingToBook = txForSendingCoinToBook
                .Outputs
                .AsCoins()
                .First()
                .ToScriptCoin(redeemScriptPubKeyForSendingCoinToBook);
```  

So let’s create a transaction that want to spend such output:  

```cs
//Create spending transaction.
Transaction txSpendingCoinOfThisBook = new Transaction();
txSpendingCoinOfThisBook.AddInput(new TxIn(new OutPoint(txForSendingCoinToBook, 0)));
```  

The first option for any spender to spend the coin is to know my birth date to prove the ownership for the coin by interacting with the scriptSig. 

```cs
//Option 1 : Spender knows my birth date.
Op pushBirthdate = Op.GetPushOp(birthDate);
//Go to IF.
Op selectIf = OpcodeType.OP_1; 
Op redeemBytes = Op.GetPushOp(redeemScriptPubKeyForSendingCoinToBook.ToBytes());
Script scriptSig = new Script(pushBirthdate, selectIf, redeemBytes);
txSpendingCoinOfThisBook.Inputs[0].ScriptSig = scriptSig;
```  

You can see that in the **scriptSig** I push **OP_1** so I enter in the **OP_IF** of my **RedeemScript**.  
Since there is no backed-in template, for creating such **scriptSig**, you can see how to build a P2SH **scriptSig** by hand.

You can see that in the **scriptSig** I push **OP_1** so I enter into my **redeemScriptPubKeyForSendingCoinToBook".  

```cs
//Verify the script pass
var verificationByBirthDate = txSpendingCoinOfThisBook
                .Inputs
                .AsIndexedInputs()
                .First()
                .VerifyScript(txForSendingCoinToBook.Outputs[0].ScriptPubKey);
Console.WriteLine(result);
//Output:
//True
```  

The second way for some spender to spend the coin is by proving ownership of the Bitcoin address, **1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB**, by corresponding the private key.  
```
//Option 2 : Spender knows my private key.
BitcoinSecret secret = new BitcoinSecret("PrivateKeyRepresentedInBase58StringRelatedToTheBookBitcoinAddress");
var sig = txSpendingCoinOfThisBook.SignInput(privateKeyRelatedToTheBookBitcoinAddress, scriptCoinForSendingToBook);
var p2pkhProof = PayToPubkeyHashTemplate
    .Instance
    .GenerateScriptSig(sig, privateKeyRelatedToTheBookBitcoinAddress.PrivateKey.PubKey);
//Go to IF.
selectIf = OpcodeType.OP_0; 
scriptSig = p2pkhProof + selectIf + redeemBytes;
txSpendingCoinOfThisBook.Inputs[0].ScriptSig = scriptSig;
```  

And ownership is also proven:  

```cs
//Verify the script pass
var verificationByPrivateKey = txSpendingCoinOfThisBook
                .Inputs
                .AsIndexedInputs()
                .First()
                .VerifyScript(txForSendingCoinToBook.Outputs[0].ScriptPubKey);
Console.WriteLine(verificationByPrivateKey);
//Output:
//True
```  
