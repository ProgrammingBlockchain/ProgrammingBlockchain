## Arbitrary {#arbitrary}

Dari versi Bitcoin 0.10, **RedeemScript** telah bisa _arbitrary_, artinya adalah bahwa dengan bahasa script Bitcoin, anda bisa membuat sendiri definisi “kepemilikan \(ownership\)”.

Contohnya, Saya bisa memberi uang kepada siapapun yang mengetahui tanggal lahir saya \(dd\/mm\/yyyy\) dalam UTF8, atau yang tahu private key dari **1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB**.

Detail bahasa script berasal dari luar, anda dapat dengan mudah mencari di berbagai website, dan merupakan bahasa berbasis stack, sehingga orang yang telah melakukan assembler dapat membacanya.

> **Catatan:** \([nopara73](https://github.com/nopara73)\) Saya menemukan tutorial [Davide De Rosa](http://davidederosa.com/basic-blockchain-programming/bitcoin-script-language-part-one/),salah satu yang paling menarik.

Pertama, membuat **RedeemScript**,

> **Catatan:** Agar kode berikut dapat bekerja, klik kanan **References** -&gt;** Add Reference...** -&gt; Cari **System.Numerics**

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

**RedeemScript** ini artinya akan ada dua cara pengeluaran seperti di **ScriptCoin**: baik anda yang mengetahui data tanggal lahir \(**birthHash**\), ataupun juga anda yang memiliki address bitcoinnya.

Jadi katakan saja saya akan mengirimkan uang seperti dalam **redeemScript **berikut:

```cs
var tx = new Transaction();
tx.Outputs.Add(new TxOut(Money.Parse("0.0001"), redeemScript.Hash));
ScriptCoin scriptCoin = tx.Outputs.AsCoins().First().ToScriptCoin(redeemScript);
```

Jadi kita akan membuat transaksi pengeluaran seperti ini:

```cs
//Create spending transaction
Transaction spending = new Transaction();
spending.AddInput(new TxIn(new OutPoint(tx, 0)));
```

Opsi pertama adalah untuk mengetahui tanggal lahir saya, dan untuk dapat membuktikannya di **scriptSig**:

```cs
////Option 1 : Spender knows my birthdate
Op pushBirthdate = Op.GetPushOp(birth);
Op selectIf = OpcodeType.OP_1; //go to if
Op redeemBytes = Op.GetPushOp(redeemScript.ToBytes());
Script scriptSig = new Script(pushBirthdate, selectIf, redeemBytes);
spending.Inputs[0].ScriptSig = scriptSig;
```

Anda dapat melihat di **scriptSig, **saya mendorong **OP\_1, **jadi saya memasukkan dalam **OP\_IF** dari **RedeemScript**.  
Karena tiedak ada template yang mendukung, untuk membuat **scriptSig**, anda dapat melihat bagaimana membuat sebuah **scriptSig** P2SH.

Kemudian anda juga dapat memeriksa bahwa **scriptSig** dapat membuktikan kepemilikan **scriptPubKey**:

```cs
//Verify the script pass
var result = spending
                .Inputs
                .AsIndexedInputs()
                .First()
                .VerifyScript(tx.Outputs[0].ScriptPubKey);
Console.WriteLine(result); // True
```

Cara yang kedua adalah untuk membuktikan kepemilikan address **1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB**.

```
////Option 2 : Spender knows my private key
BitcoinSecret secret = new BitcoinSecret("...");
var sig = spending.SignInput(secret, scriptCoin);
var p2pkhProof = PayToPubkeyHashTemplate
    .Instance
    .GenerateScriptSig(sig, secret.PrivateKey.PubKey);
selectIf = OpcodeType.OP_0; //go to else
scriptSig = p2pkhProof + selectIf + redeemBytes;
spending.Inputs[0].ScriptSig = scriptSig;
```

Dan kepemilikan tersebut juga dapat dibuktikan:

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

