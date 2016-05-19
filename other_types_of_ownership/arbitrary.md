## Arbitrary {#arbitrary}

From Bitcoin 0.10, the **RedeemScript** can be arbitrary, which means that with the script language of Bitcoin, you can create your own definition of what “ownership” means.

For example, I can give money to whoever either know my date of birth (dd/mm/yyyy) serialized in UTF8 either knows the private key of **1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB**.

The details of the script language are out of scope, you can easily find the documentation on various websites, and it is a stack based language so everyone having done some assembler should be able to read it.  

> **Note:** ([nopara73](https://github.com/nopara73)) I find [Davide De Rosa's tutorial](http://davidederosa.com/basic-blockchain-programming/bitcoin-script-language-part-one/) as the most enjoyable one.

So first, let’s build the **RedeemScript**,  

> **Note:** For this code to work right click **References** ->** Add Reference...** -> Find **System.Numerics**

```cs
BitcoinAddress address = BitcoinAddress.Create("1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB");
var birth = Encoding.UTF8.GetBytes("18/07/1988");
var birthHash = Hashes.Hash256(birth);
Script redeemScript = new Script(
    "OP_IF "
        + "OP_HASH256 " + Op.GetPushOp(birthHash.ToBytes()) + " OP_EQUAL " +
    "OP_ELSE "
        + address.ScriptPubKey + " " +
    "OP_ENDIF");
```

This **RedeemScript** means that there is 2 way of spending such **ScriptCoin**: either you know the data that give **birthHash** (my birthdate), either you own the bitcoin address.

Let’s say I sent money to such **redeemScript**:

So let’s create a transaction that want to spend such output:

The first option is to know my birth date and to prove it in the **scriptSig**:

You can see that in the **scriptSig** I push **OP_1** so I enter in the **OP_IF** of my **RedeemScript**.Since there is no backed-in template, for creating such **scriptSig**, you can see how to build a P2SH **scriptSig** by hand.

Then you can check that the **scriptSig** prove the ownership of the **scriptPubKey**:

True

The second way of spending the coin is by proving ownership of **1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB**.

And ownership is also proven

True