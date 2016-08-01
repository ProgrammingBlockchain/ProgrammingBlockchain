## P2W\* diatas P2SH {#p2w-over-p2sh}

Saat menggunakan **witness scriptPubKey** dalam scripting anda, masih ada kebutuhan yang harus terpenuhi, karena kenyataannya sebagaian besar wallet saat ini hanya support address P2PKH atau P2SH.

Untuk dapat memanfaatkan keuntungan dari segwit, masih kompatibel juga pada software lama, seperti pada P2W atau P2SH. Node lama, akan membacanya seperti pada pembayaran normal P2SH.

Anda dapat mengubah setiap **P2W\*** menjadi **P2W\* over** **P2SH** dengan cara:

1. Mengganti **ScriptPubKey** dengan P2SH.
2. **ScriptPubKey** yang telah digantikan, ditempatkan sebagai satu-satunya dorongan dalam **scriptSig** untuk transaksi pengeluaran,
3. Semua data lain akan menjadi pendorong dalam witness pada transaksi pengeluaran.

Agak rumit memang, namun jangan khawatir, karena TransactionBuilder dapat memudahkan abstraksi pembuatannya menjadi lebih efektif.

Mari kita ambil contoh pada P2WPKH melalui P2SH, atau yang disebut dengan **P2SH\(P2WPKH\)**.

Printing **ScriptPubKey**:

```cs
var key = new Key();
Console.WriteLine(key.PubKey.WitHash.ScriptPubKey.Hash.ScriptPubKey);
```

> **Catatan:** Kode diatas cukup mengagumkan.

Yang membuat **scriptPubKey **P2SH banyak dikenal.

```
OP_HASH160 b19da5ca6e7243d4ec8eab07b713ff8768a44145 OP_EQUAL
```

Lalu, menandatangani output transaksi pengeluaran seperti ini:

```json
"in": [
    {
      "prev_out": {
        "hash": "674ece694e5e28956138efacab96fc0bffd7c6cc1af7bb2729943fedf8f0b8b9",
        "n": 0
      },
      "scriptSig": "001404100ab485c95701bf0f4d73e3fe7d69ecc4f0ea",
      "witness": "3045022100f4c14cf383c0c97bbdaf520ea06f7db6c61e0effbc4bd3dfea036a90272f6cce022055b0fc058759a7961e718d48a3dc4dd5580fffc310557925a0865dbe467a835901 0205b956a5afe8f34a01337f0949f5733b5e376caaea57c9624e40e739a0b1d16c"
    }
  ],
```

**ScriptSig** hanya menekan script redem P2SH pada ScriptPubKey sebelumnya \(atau **key.PubKey.WitHash.ScriptPubKey**\).Witness sama seperti pembayaran **P2WPKH**.

Dalam NBitcoin, penandatanganan sebuah P2SH\(P2WPKH\) sama persis seperti penandatanganan P2SH secara normal dengan ScriptCoin.

Dengan prinsip yang sama, kita coba melihat bagaimana sebuah **P2SH\(P2WSH\)** akan terlihat. Anda perlu memahami bahwa dalam hal ini, kita berhadapan pada dua redem sript yang berbeda: **P2SH redeem script** yang dimasukkan kedalam **scriptSig** dari transaksi pengeluaran, dan juga **P2WSH redeem script** yang dimasukkan ke dalam witness.

**ScriptPubKey** pada aturan yang pertama:

1. Mengganti **ScriptPubKey** dengan P2SH yang setara. 

  ```cs
  var key = new Key();
  Console.WriteLine(key.PubKey.ScriptPubKey.WitHash.ScriptPubKey.Hash.ScriptPubKey);
  ```

  ```
  OP_HASH160 d06c0058175952afecc56d26ed16558b1ed40e42 OP_EQUAL
  ```

  > **Warning:** It makes sense, don't try whiny ragequitting!

2. **ScriptPubKey** yang telah digantikan, akan ditempatkan untuk mendorong **scriptSig** pada transaksi pengeluaran,

3. Semua data lain akan jadi pendorong witness dalam transaksi pengeluaran,


Untuk yang ke-3. Pada **‘data lain’ **tersebut, dalam konteks pembayaran P2WSH berarti menjadi parameter dari **P2WSH redeem script, **diikuti dengan dorongan sebuah **P2WSH redeem script**.

```json
  "in": [
    {
      "prev_out": {
        "hash": "1d23fa744a26cf6433f0841e9de7e088cf95e6f953e584b98d0de6ef4216765f",
        "n": 0
      },
      "scriptSig": "0020c54eb79829b2e26b71d15fd3b490b6e95cbdab361a45eed2cdfe642497480a6c",
      "witness": "3045022100d7570c3bf87149a0be3ba2e8bfccbdd35c3da44f741695e9962014795fabc4fc02203183cfa55a85728520b0f1ac59ac3ffa1a8526634fe619f99fac0f76016f366e01 2103146e87d7fcc81f3e044f97c6b262c01826f40a9ab9acae0f689983a5890a1f4dac"
    }
  ],

```

Singkatnya, script redem P2SH di hash untuk mendapat scriptPubKey P2WSH sebagai pembayaran normal P2WSH. Kemudian, sebagai sebuah pembayaran normal P2SH, scriptPubKey P2WSH digantikan oleh hash dan digunakan untuk membuat P2SH yang sebenarnya.

Jika P2SH\/P2WSH\/P2SH\(P2WSH\)\/P2SH\(P2WPKH\) nampak cukup rumit, jangan khawatir.  
NBitcoin, untuk **semua tipe pembayaran** tersebut, anda hanya membutuhkan membuat sebuah **ScriptCoin** dengan menerapkan redem script \(script redem P2WSH atau P2SH\) dan ScriptPubKey, sama persis seperti yang telah dijelaskan pada pembahasan **P2SH**.

Pada NBitcoin, anda hanya perlu menentukan output transaksi pengeluaran, dengan redem script yang tepat. Sementara pada **TransactionBuilder** akan mencari cara bagaimana menandatanganinya dengan tepat sama halnya yang telah dijelaskan pada pembahasan **Multi Sig. **Selanjutnya dibahas di pembahasan “**Penggunaan TransactionBuilder**”.

![](../assets/ScriptCoin.png)

**Kompatibel untuk P2SH\/P2WSH\/P2SH\(P2WSH\)\/P2SH\(P2WPKH\)**

Anda dapat mencari contoh pembayaran P2W\* di [http:\/\/n.bitcoin.ninja\/checkscript](http://n.bitcoin.ninja/checkscript)

