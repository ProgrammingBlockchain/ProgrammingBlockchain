## Transaction {#transaction}

> ([Mastering Bitcoin](https://github.com/bitcoinbook/bitcoinbook/)) Transactions are the most important part of the bitcoin system. Everything else in bitcoin is designed to ensure that transactions can be created, propagated on the network, validated, and finally added to the global ledger of transactions (the blockchain). Transactions are data structures that encode the transfer of value between participants in the bitcoin system. Each transaction is a public entry in bitcoin’s blockchain, the global double-entry bookkeeping ledger.

A transaction may have no recipient, or it may have several. **The same can be said for senders!** On the Blockchain, the sender and recipient are always abstracted with a ScriptPubKey, as we demonstrated in previous chapters.  

If you use Bitcoin Core your Transactions tab will show the transaction, like this:

![](../assets/BitcoinCoreTransaction.png)  

For now we are interested in the **Transaction ID**. In this case, it is ```f13dc48fb035bbf0a6e989a26b3ecb57b84f85e0836e777d6edf60d87a4a2d94```  

> **Note:** The TransactionId is defined by SHA256(SHA256(txbytes))

> **Note:** Do NOT use the TransactionId to handle unconfirmed transactions. The TransactionId can be manipulated before it is confirmed. This is known as “Transaction Malleability.”

You can review the transaction on a blockexplorer like Blockchain.info: https://blockchain.info/tx/f13dc48fb035bbf0a6e989a26b3ecb57b84f85e0836e777d6edf60d87a4a2d94 
But as a developer you will probably want a service that is easier to query and parse.  
As a C# developer and an NBitcoin user Nicolas Dorier's [QBit Ninja](http://docs.qbitninja.apiary.io/) will definitely be your best choice. It is an open source web service API to query the blockchain and for tracking wallets.  
QBit Ninja depends on [NBitcoin.Indexer](https://github.com/MetacoSA/NBitcoin.Indexer) which rely on Microsoft Azure Storage. C# developers are expected to use the [NuGet client package](http://www.nuget.org/packages/QBitninja.Client) instead of developping a wrapper around this API.  

If you go to http://api.qbit.ninja/transactions/f13dc48fb035bbf0a6e989a26b3ecb57b84f85e0836e777d6edf60d87a4a2d94 you will see the raw bytes of your transaction.  

![](../assets/RawTx.png)  

Quickly close the tab, before it scares you away from the computer. QBit Ninja queries the API and parses the information for you so you don’t have to do it manually.  
Go ahaead and install **QBitNinja.Client** NuGet package.  

![](../assets/QBitNuGet.png)  

Now let's see the code:

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