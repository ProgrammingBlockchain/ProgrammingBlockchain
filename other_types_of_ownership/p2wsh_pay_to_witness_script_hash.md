## P2WSH \(Pay to Witness Script Hash\) {#p2wsh-pay-to-witness-script-hash}

Hampir sama seperti pada P2PKH\/P2WPKH, yang membedakan antara P2SH dan P2WSH adalah pada penempatannya saja Seperti yang sebelumnya di `scriptSig`, dan `scriptPubKey` yang telah dimodifikasi.

Pada `scriptPubKey` yang akan dimodifikasi terlihat seperti ini:

`OP_HASH160 10f400e996c34410d02ae76639cbf64f4bdf2def OP_EQUAL`

Menjadi:

`0 e4d3d21bab744d90cd857f56833252000ac0fade318136b713994b9319562467`

Untuk mencetaknya bisa dengan menggunakan kode ini:

```cs
var key = new Key();
Console.WriteLine(key.PubKey.ScriptPubKey.WitHash.ScriptPubKey);
```

Dengan `scriptSig` yang ada sebelumnya \(signature + redeem script\), dipindahkan ke `witness`:

```json
"in": [
    {
      "prev_out": {
        "hash": "ffa2826ba2c9a178f7ced0737b559410364a62a41b16440beb299754114888c4",
        "n": 0
      },
      "scriptSig": "",
      "witness": "304402203a4d9f42c190682826ead3f88d9d87e8c47db57f5c272637441bafe11d5ad8a302206ac21b2bfe831216059ac4c91ec3e4458c78190613802975f5da5d11b55a69c601 210243b3760ce117a85540d88fa9d3d605338d4689bed1217e1fa84c78c22999fe08ac"
    }
  ]

```

Seperrti pembayaran P2SH yang telah dijelaskan sebelumnya, P2WSH menggunakan cara yang sama pada `ScriptCoin` untuk penandatanganan transaksi.

