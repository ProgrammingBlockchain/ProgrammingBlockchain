## Spend your coin {#spend-your-coin}

So now that you know what a **bitcoin address**, a **ScriptPubKey**, a **private key**, and a **miner** are you will make your first **transaction** by hand.  

As you proceed through this lesson you will add code line by line as it is presented to build a method that will leave feedback for the book in a Twitter style message.  

Let’s start by looking at the **transaction** that contains the **TxOut** that you want to spend as we did previously:  

Create a new **Console Project** (>.net45) and install **QBitNinja.Client** NuGet.  

Have you already generated and noted a private key to yourself? Have you already get the corresponding bitcoin address and sent some funds there? If not, don't worry, I quickly reiterate how you can do it:  

```cs
var network = Network.Main;

Key privateKey = new Key();
var bitcoinPrivateKey = privateKey.GetWif(network);
var address = bitcoinPrivateKey.GetAddress();

Console.WriteLine(bitcoinPrivateKey);
Console.WriteLine(address);
```  

Note the **bitcoinPrivateKey**, the **address**, send some coins there and note the transaction id (you can find it (probably) in your wallet software or with a blockexplorer, like [blockchain.info](http://blockchain.info/)).  

Import your private key:  

```cs
var bitcoinPrivateKey = new 
BitcoinSecret("cSZjE4aJNPpBtU6xvJ6J4iBzDgTmzTjbq8w2kqnYvAprBCyTsG4x");
var network = bitcoinPrivateKey.Network;
var address = bitcoinPrivateKey.GetAddress();

Console.WriteLine(bitcoinPrivateKey); // cSZjE4aJNPpBtU6xvJ6J4iBzDgTmzTjbq8w2kqnYvAprBCyTsG4x
Console.WriteLine(address); // mzK6Jy5mer3ABBxfHdcxXEChsn3mkv8qJv
```  

And finally get the transaction info:  
```cs
var client = new QBitNinjaClient(network);
var transactionId = uint256.Parse("e44587cf08b4f03b0e8b4ae7562217796ec47b8c91666681d71329b764add2e3");
var transactionResponse = client.GetTransaction(transactionId).Result;

Console.WriteLine(transactionResponse.TransactionId); // e44587cf08b4f03b0e8b4ae7562217796ec47b8c91666681d71329b764add2e3
Console.WriteLine(transactionResponse.Block.Confirmations);
```  

Now we have every information for creating our transactions. The main questions are: **from where, to where and how much?**

### From where?

In our case, we want to spend the second outpoint. Here's how we have figured this out:  
```cs
var receivedCoins = transactionResponse.ReceivedCoins;
OutPoint outPointToSpend = null;
foreach (var coin in receivedCoins)
{
    if (coin.TxOut.ScriptPubKey == bitcoinPrivateKey.ScriptPubKey)
    {
        outPointToSpend = coin.Outpoint;
    }
}
if(outPointToSpend == null)
    throw new Exception("TxOut doesn't contain our ScriptPubKey");
Console.WriteLine("We want to spend {0}. outpoint:", outPointToSpend.N + 1);
```  

For the payment you will need to reference this outpoint in the transaction. You create a transaction as follows:

```cs
var transaction = new Transaction();
transaction.Inputs.Add(new TxIn()
{
    PrevOut = outPointToSpend
});
```  

### To where?

Do you remember the main questions? **From where, to where and how much?**  
Constructing **TxIn** and adding to the transaction was the answer the "from where" question.  
Constructing **TxOut** and adding to the transaction is the answer for the remaining ones.  

The donation address of this book is: [1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB](https://blockchain.info/address/1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB)  
This money goes into my "Coffee and Sushi Wallet" that will keep me fed and compliant while writing the rest of the book.  
If you succeed to complete this challange you will be able to find your contribution among **Hall of the Makers** on http://n.bitcoin.ninja/ (ordered by generosity).  
```cs
var hallOfTheMakers = new BitcoinPubKeyAddress("1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB");
```  
If you are working on the testnet, send the testnet coins to any testnet address.
```cs
var hallOfTheMakers = new BitcoinPubKeyAddress("mzp4No5cmCXjZUpf112B1XWsvWBfws5bbB");
```  

### How much?
If you want to send **0.5 BTC** from a transaction input with **1 BTC** you actually have to spend all!   
As the diagram shows below, your transaction output specifies  **0.5** BTC to Hall of The Makers and **0.4999** back to you.  
What happens to the remaining **0.0001 BTC**? This is the miner fee in order to incentivize them to add this transaction into their next block.

![](../assets/SpendTx.png)  

You can check the address I am working with on my example (I am working on the testnet): http://tbtc.blockr.io/address/info/mzK6Jy5mer3ABBxfHdcxXEChsn3mkv8qJv  


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