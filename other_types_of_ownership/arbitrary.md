## Arbitrary {#arbitrary}

From Bitcoin 0.10, the **RedeemScript** can be arbitrary, which means that with the script language of Bitcoin, you can create your own definition of what “ownership” means.

For example, I can give money to whoever knows either my date of birth (dd/mm/yyyy) serialized in UTF-8 or the private key of **1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB**.

The details of the script language are out of scope. You can easily find the documentation on various websites. The Bitcoin script language is a stack based language so everyone having done some assembler should be able to read it.  

> **Note:** ([nopara73](https://github.com/nopara73)) I find [Davide De Rosa's tutorial](http://davidederosa.com/basic-blockchain-programming/bitcoin-script-language-part-one/) as the most enjoyable one.

So first, let’s build the **RedeemScript**,  

> **Note:** For this code to work right click **References** ->** Add Reference...** -> Find **System.Numerics**

```cs
BitcoinAddress address = BitcoinAddress.Create("1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB", Network.Main);
var birth = Encoding.UTF8.GetBytes("18/07/1988");
var birthHash = Hashes.DoubleSHA256(birth);
Script redeemScript = new Script(
    "OP_IF "
        + "OP_HASH256 " + Op.GetPushOp(birthHash.ToBytes()) + " OP_EQUAL " +
    "OP_ELSE "
        + address.ScriptPubKey + " " +
    "OP_ENDIF");
```

This **RedeemScript** means that there are 2 ways of spending such **ScriptCoin**: Either you know the data that gives **birthHash** (my birthdate) or you own the bitcoin address.

Let’s say I sent money to such **redeemScript**:  

```cs
var tx = Network.Main.CreateTransaction();
tx.Outputs.Add(Money.Parse("0.0001"), redeemScript.Hash);
ScriptCoin scriptCoin = tx.Outputs.AsCoins().First().ToScriptCoin(redeemScript);
```  

So let’s create a transaction that wants to spend such output:  

```cs
//Create spending transaction
Transaction spending = Network.Main.CreateTransaction();
spending.AddInput(new TxIn(new OutPoint(tx, 0)));
```  

The first option is to know my birth date and to prove it in the **scriptSig**:  

```cs
////Option 1 : Spender knows my birthdate
Op pushBirthdate = Op.GetPushOp(birth);
Op selectIf = OpcodeType.OP_1; //go to if
Op redeemBytes = Op.GetPushOp(redeemScript.ToBytes());
Script scriptSig = new Script(pushBirthdate, selectIf, redeemBytes);
spending.Inputs[0].ScriptSig = scriptSig;
```  

You can see that in the **scriptSig** I push **OP_1** so I enter in the **OP_IF** of my **RedeemScript**.  
Since there is no backed-in template, for creating such **scriptSig**, you can see how to build a P2SH **scriptSig** by hand.

Then you can check that the **scriptSig** proves the ownership of the **scriptPubKey**:  

```cs
//Verify the script pass
var result = spending
                .Inputs
                .AsIndexedInputs()
                .First()
                .VerifyScript(tx.Outputs[0].ScriptPubKey);
Console.WriteLine(result); // True
```  

The second way of spending the coin is by proving ownership of **1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB**.  
```cs
////Option 2 : Spender knows my private key
BitcoinSecret secret = new BitcoinSecret("...", Network.Mainnet);
var sig = spending.SignInput(secret, scriptCoin);
var p2pkhProof = PayToPubkeyHashTemplate
    .Instance
    .GenerateScriptSig(sig, secret.PrivateKey.PubKey);
selectIf = OpcodeType.OP_0; //go to else
scriptSig = p2pkhProof + selectIf + redeemBytes;
spending.Inputs[0].ScriptSig = scriptSig;
```  

And ownership is also proven:  

```cs
//Verify the script pass
result = spending
                .Inputs
                .AsIndexedInputs()
                .First()
                .VerifyScript(tx.Outputs[0].ScriptPubKey);
Console.WriteLine(result); // True
///////////
```  
