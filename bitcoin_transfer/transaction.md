## Transaction {#transaction}

Before we begin, remember to create a new chapter class.

A **transaction** is a transfer of bitcoin. A transaction may have no recipient, or it may have several. The same can be said for senders! On the Blockchain, the sender and recipient are always abstracted with a scriptPubKey, as we demonstrated in Chapter 10\. We will move forward under the assumption that you’ve completed Lesson 4 and the associated exercises. If you have not completed the exercises, please send money to the address that you generated before continuing.

If you used Bitcoin Core your transactions tab will show the transaction, like this:

For now we’re interested in the **Transaction** ID. In this case, it’s f13dc48fb035bbf0a6e989a26b3ecb57b84f85e0836e777d6edf60d87a4a2d94

The TransactionId is defined by SHA256(SHA256(txbytes))

Do NOT use the TransactionId to handle unconfirmed transactions. The TransactionId can be manipulated before it is confirmed. This is known as “Transaction Malleability.”

You can review the transaction on a website like Blockchain.info, but as a developer you will probably want a service that is easier to query and parse. At the time of this writing, we find Blockr.io to be a good service for the task.

If you go to http://btc.blockr.io/api/v1/tx/raw/f13dc48fb035bbf0a6e989a26b3ecb57b84f85e0836e777d6edf60d87a4a2d94 you will see the raw bytes of your transaction.

NBitcoin queries blockr and parses the information for you so you don’t have to do it manually.

"hash": "4ebf7f7ca0a5dafd10b9bd74d8cb93a6eb0831bcb637fec8e8aabf842f1c2688",

"ver": 1,

"vin_sz": 1,

"vout_sz": 2,

"lock_time": 0,

"size": 225,

"in": [

{

"prev_out": {

"hash": "bf7d91ac70917f98b497927e1b07267507652b206df14ecdba2e9390b9bffc65",

"n": 0

},

"scriptSig": "3044022069b6b0f1a8d453bdb89e3ad475232b8e01d2851e7b53acab3f830f40e80b3b5102203c049867975360020293c735d48b4a2dda003aa781c1d8ccd2c7af290dcd11de01 02e3538427350039e67ea99e935cefb740badf3d09ebc301b0bc9d1bb0301a3417"

}

],

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

The relevant parts for now are **in** and **out**. You can see that in out 0.0899 Bitcoin has been sent to a scriptPubKey, and 0.01 has been sent to another. (**Exercise**: Verify the public key hash in this ScriptPubKey is the same as the one associated with your paymentAddress)

If you look at the **in** you will notice a **prev_out** (previous **out**) is referenced. Each in show you which previous out has been spent in order to fund this transaction. The terms **TxOut** and **Output** are synonymous with out.

In summary, the TxOut represents an amount of bitcoin and a **ScriptPubKey**. (Recipient)

Every outhas an address defined by the transaction ID andindex called the **Outpoint**. For example, the Outpoint of the out with 0.01 BTC in my transaction is (71049fd47ba2107db70d53b127cae4ff0a37b4ab, 1).

Now let’s take a look at the in (aka **TxIn**, **Inputs**) of the transaction:

The TxIn is composed of the Outpoint of the prev_out being spent and of a **ScriptSig** also called “Proof of Ownership.” In my case, the prev_out Outpoint is (7def8a69a7a2c14948f3c4b9033b7b30f230308b, 0)

By replacing the transaction ID in the code we wrote for Lesson1 we can review the information associated with that transaction. We could continue to trace the transaction IDs back in this manner until we reach the bitcoins’ **coinbase,** the block where they were mined.

In our example, the prev_out was for a total of .1 BTC. In this transaction .0899 BTC and .01 BTC were sent. That means 0.0001 BTC (or 0.1 - (0.0899+0.01)) is not accounted for! The difference between the inputs and outputs are called **Transaction Fees** or **Miner’s Fees**. This is the money that the miner collects for including a given transaction in a block.