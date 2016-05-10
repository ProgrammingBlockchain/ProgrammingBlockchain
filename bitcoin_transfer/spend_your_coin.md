## Spend your coin {#spend-your-coin}

So now that you know what a bitcoin address, a ScriptPubKey, a private key, and a miner are you’ll make your first transaction by hand. Create a new class called Chapter14 and a method called Lesson1\. As you proceed through this chapter you will add code line by line as it is presented to build a method that will leave feedback for the book in a Twitter style message.

Let’s start by looking at the transaction that that contains the TxOut that you want to spend as we did in Chapter 11\.

In our case, we want to spend the second output:

"out": [

{

"value": "0.08990000",

"scriptPubKey": "OP_DUP OP_HASH160 5b1d720daf0e95e37d0eaedd282b6ed9a40bab71 OP_EQUALVERIFY OP_CHECKSIG"

},

{

"value": "0.01000000",

"scriptPubKey": "OP_DUP OP_HASH160 71049fd47ba2107db70d53b127cae4ff0a37b4ab OP_EQUALVERIFY OP_CHECKSIG"

}

]

For the payment you will need to reference this output in the transaction. You create a transaction as follows:

Now let’s take care about the Outputs. We as that you send **0.004 BTC**, and since you spent **0.01 BTC**, you want **0.006 BTC** back. You will also give some fees to the miners to incentivize them to add this transaction into their next block. So you will take back **0.0059 BTC**.

The book’s donation address is: 1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB

Now add your feedback! This must be less than 40 bytes, or it will crash the application.

{

"hash": "258ed68ac5a813fe95a6366d94701314f59af1446dda2360cf6f8e505e3fd1b6",

"ver": 1,

"vin_sz": 1,

"vout_sz": 3,

"lock_time": 0,

"size": 166,

"in": [

{

"prev_out": {

"hash": "4ebf7f7ca0a5dafd10b9bd74d8cb93a6eb0831bcb637fec8e8aabf842f1c2688",

"n": 1

},

"scriptSig": ""

}

],

"out": [

{

"value": "0.00400000",

"scriptPubKey": "OP_DUP OP_HASH160 c81e8e7b7ffca043b088a992795b15887c961592 OP_EQUALVERIFY OP_CHECKSIG"

},

{

"value": "0.00590000",

"scriptPubKey": "OP_DUP OP_HASH160 71049fd47ba2107db70d53b127cae4ff0a37b4ab OP_EQUALVERIFY OP_CHECKSIG"

},

{

"value": "0.00000000",

"scriptPubKey": "OP_RETURN 42696c6c20537472616974206973207570646174696e672073637265656e73686f74732e"

}

]

}

Now that we have created the transaction, we must sign it. In other words, you will have to prove that you own the TxOut that you referenced in the input.

Signing can be complicated. Refer to [https://en.bitcoin.it/w/images/en/7/70/Bitcoin_OpCheckSig_InDetail.png](https://en.bitcoin.it/w/images/en/7/70/Bitcoin_OpCheckSig_InDetail.png) for details. But we’ll make it simple.

First insert the scriptPubKey in the **scriptSig.**Since the scriptPubKey is nothing but **paymentAddress.ScriptPubKey** this is simple.Then you need to give your private key for signing.

"in": [

{

"prev_out": {

"hash": "4ebf7f7ca0a5dafd10b9bd74d8cb93a6eb0831bcb637fec8e8aabf842f1c2688",

"n": 1

},

"scriptSig": "3045022100d4d8d4e1e3205399e0ab899e95bd6c7b65a094c9b86c4b020872428864a5c63502206f9d0b3084e4a7895de520069a939347909471ca118a43723fd8734cd8e8bcac01 03e0b917b43bb877ccb68c73c82165db6d01174f00972626c5bea62e64d57dea1a"

}

]

Congratulations, you have signed your first transaction! Your transaction is ready to roll! All that is left is to propagate it to the network so the miners can see it. Be sure to have Bitcoin Core running and then:

The **using** code block will take care of closing the connection to the node. That it!

You can also connect directly to the Bitcoin network, However I advise you to connect to your own trusted node (faster and easier)