## P2WSH (Pay to Witness Script Hash) {#p2wsh-pay-to-witness-script-hash}

As with P2PKH/P2WPKH, the only difference between P2SH and P2WSH is about the location of what was previously in the ```scriptSig```, and the ```scriptPubKey``` being modified.

The ```scriptPubKey``` is changed from something like:

```OP_HASH160 10f400e996c34410d02ae76639cbf64f4bdf2def OP_EQUAL```

To:

```0 e4d3d21bab744d90cd857f56833252000ac0fade318136b713994b9319562467```

That you can print with the following code:  

```cs
var key = new Key();
Console.WriteLine(key.PubKey.ScriptPubKey.WitHash.ScriptPubKey);
```  

With what was previously in the ```scriptSig``` (signature + redeem script), moved to the ```witness```:

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

As the P2SH payment explained previously, P2WSH uses ```ScriptCoin``` in exactly the same way to be signed.
