## Transfer an Asset {#transfer-an-asset}

So now, let’s imagine I sent you some **BlockchainProgramming** Coins.How can you send me back the coins so I unlock the part 3 just for you?You need to build a **ColoredCoin**.

In the sample above, let’s say I want to spend the 10 assets I received on the address “nico”.From this [web service](http://rapidbase-test.azurewebsites.net/balances/akFqRqfdmAaXfPDmvQZVpcAQnQZmqrx4gcZ), I can see what coin I want to spend.

{

"transactionId": "fa6db7a2e478f3a8a0d1a77456ca5c9fa593e49fd0cf65c7e349e5a4cbe58842",

"index": 0,

"value": 600,

"scriptPubKey": "76a914356facdac5f5bcae995d13e667bb5864fd1e7d5988ac",

"redeemScript": null,

"assetId": "AVAVfLSb1KZf9tJzrUVpktjxKUXGxUTD4e",

"quantity": 10

}

Here is how to instantiate such Colored Coin in code:

We’ll show later how you can use some web services or custom code to get the coins more easily. I also needed another coin (forFees), to pay the fees.The asset transfer is actually very easy with the **TransactionBuilder**.

{

….

"out": [

{

"value": "0.00000000",

"scriptPubKey": "OP_RETURN 4f410100010a00"

},

{

"value": "0.00000600",

"scriptPubKey": "OP_DUP OP_HASH160 c81e8e7b7ffca043b088a992795b15887c961592 OP_EQUALVERIFY OP_CHECKSIG"

},

{

"value": "0.04415000",

"scriptPubKey": "OP_DUP OP_HASH160 356facdac5f5bcae995d13e667bb5864fd1e7d59 OP_EQUALVERIFY OP_CHECKSIG"

}

]

}

Which basically succeed: