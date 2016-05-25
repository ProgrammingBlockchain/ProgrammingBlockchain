## Spend your coin {#spend-your-coin}

So now that you know what a **bitcoin address**, a **ScriptPubKey**, a **private key**, and a **miner** are you will make your first **transaction** by hand.  

As you proceed through this lesson you will add code line by line as it is presented to build a method that will leave feedback for the book in a Twitter style message.  

Let’s start by looking at the **transaction** that contains the **TxOut** that you want to spend as we did previously:  

Create a new **Console Project** (>.net45) and install **QBitNinja.Client** NuGet.  

Have you already generated and noted a private key to yourself? Have you already get the corresponding bitcoin address and sent some funds there? If not, don't worry, I quickly reiterate how you can do it:  

```cs
var network = Network.Main;

var privateKey = new Key();
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
var hallOfTheMakersAddress = new BitcoinPubKeyAddress("1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB");
```  
If you are working on the testnet, send the testnet coins to any testnet address.
```cs
var hallOfTheMakersAddress = BitcoinAddress.Create("mzp4No5cmCXjZUpf112B1XWsvWBfws5bbB");
```  

### How much?
If you want to send **0.5 BTC** from a **transaction input** with **1 BTC** you actually have to spend all!   
As the diagram shows below, your **transaction output** specifies  **0.5** BTC to Hall of The Makers and **0.4999** back to you.  
What happens to the remaining **0.0001 BTC**? This is the miner fee in order to incentivize them to add this transaction into their next block.

![](../assets/SpendTx.png)  

```cs
TxOut hallOfTheMakersTxOut = new TxOut()
{
    Value = new Money((decimal)0.5, MoneyUnit.BTC),
    ScriptPubKey = hallOfTheMakersAddress.ScriptPubKey
};

TxOut changeBackTxOut = new TxOut()
{
    Value = new Money((decimal)0.4999, MoneyUnit.BTC),
    ScriptPubKey = bitcoinPrivateKey.ScriptPubKey
};

transaction.Outputs.Add(hallOfTheMakersTxOut);
transaction.Outputs.Add(changeBackTxOut);
```  

We can do some finetuning here.  
You can check the address on a blockexplorer I am working with on this whole chapter example (I am working on the testnet):   http://tbtc.blockr.io/address/info/mzK6Jy5mer3ABBxfHdcxXEChsn3mkv8qJv  

```cs
// How much you want to TO
var hallOfTheMakersAmount = new Money(0.5m, MoneyUnit.BTC);
/* At the time of writing the mining fee is 0.05usd
 * Depending on the market price and
 * On the currently advised mining fee,
 * You may consider to increase or decrease it
*/
var minerFee = new Money(0.0001m, MoneyUnit.BTC);
// How much you want to spend FROM
var txInAmount = receivedCoins[(int) outPointToSpend.N].TxOut.Amount;
Money changeBackAmount = txInAmount - hallOfTheMakersAmount - minerFee;
```

Let's add our calculated values to our TxOuts:  
```cs
TxOut hallOfTheMakersTxOut = new TxOut()
{
    Value = hallOfTheMakersAmount,
    ScriptPubKey = hallOfTheMakersAddress.ScriptPubKey
};

TxOut changeBackTxOut = new TxOut()
{
    Value = changeBackAmount,
    ScriptPubKey = bitcoinPrivateKey.ScriptPubKey
};
```  

And add them to our transaction:  
```cs
transaction.Outputs.Add(hallOfTheMakersTxOut);
transaction.Outputs.Add(changeBackTxOut);
```  

### Message on The Blockchain

Now add your feedback! This must be less than 40 bytes, or it will crash the application.  
This feedback, along with your transaction will appear (after transaction is confirmed) in the [Hall of The Makers](http://n.bitcoin.ninja/). 

```cs
var message = "nopara73 loves NBitcoin!";
var bytes = Encoding.UTF8.GetBytes(message);
transaction.Outputs.Add(new TxOut()
{
    Value = Money.Zero,
    ScriptPubKey = TxNullDataTemplate.Instance.GenerateScriptPubKey(bytes)
});
```  

To sum up take a look at my whole transaction before signing:  
I have 3 **TxOut**, 2 with **value**, 1 without **value** (with the message). You can notice the differences between the **scriptPubKey**s of the "normal" **TxOut**s and the **scriptPubKey** of the **TxOut** with the message:  

```json
{
  "hash": "b7803df4b90fd615532bcbdb3b63eb1af5a2e4ae36f29a6fbf9f57d0a1842e0a",
  "ver": 1,
  "vin_sz": 1,
  "vout_sz": 3,
  "lock_time": 0,
  "size": 154,
  "in": [
    {
      "prev_out": {
        "hash": "e44587cf08b4f03b0e8b4ae7562217796ec47b8c91666681d71329b764add2e3",
        "n": 1
      },
      "scriptSig": ""
    }
  ],
  "out": [
    {
      "value": "0.50000000",
      "scriptPubKey": "OP_DUP OP_HASH160 d3a689bc36464b9d74e1721fd321d4686eae594e OP_EQUALVERIFYOP_CHECKSIG"
    },
    {
      "value": "0.62840112",
      "scriptPubKey": "OP_DUP OP_HASH160 ce2c16edb74aef1caa6db0078af9d3a5b8fd12d1 OP_EQUALVERIFYOP_CHECKSIG"
    },
    {
      "value": "0.00000000",
      "scriptPubKey": "OP_RETURN 6e6f706172613733206c6f766573204e426974636f696e21"
    }
  ]
}
```  

Take a closer look at **TxIn**. We have **prev_out** and **scriptSig** there.  
**Exercise:** try to figure out what will be and how to get the **scriptSig** in our code before you read further!  

Let's check out the **hash** of **prev_out** in a blockexplorer: http://tbtc.blockr.io/tx/info/e44587cf08b4f03b0e8b4ae7562217796ec47b8c91666681d71329b764add2e3  
In **prev_out** **n** is 1. Since we are indexing from 0, this means I want to spend the second output of the transaction.  
In the blockexplorer we can see the corresponding address is ```mzK6Jy5mer3ABBxfHdcxXEChsn3mkv8qJv``` and I can get the scriptSig from the address like this:  

```cs
var address = BitcoinAddress.Create("mzK6Jy5mer3ABBxfHdcxXEChsn3mkv8qJv");
transaction.Inputs[0].ScriptSig = address.ScriptPubKey;
```  

### Sign your transaction

Now that we have created the transaction, we must sign it. In other words, you will have to prove that you own the TxOut that you referenced in the input.  

Signing can be [complicated](https://en.bitcoin.it/w/images/en/7/70/Bitcoin_OpCheckSig_InDetail.png), but we’ll make it simple.

First let's revisit the **scriptSig** of **in**, how we can get it from code. Remember, we copypasted the address above from a blockexplorer, now let's get it from our QBitNinja transactionResponse:  

```cs
transaction.Inputs[0].ScriptSig =  bitcoinPrivateKey.ScriptPubKey;
```  

Then you need to give your private key for signing:  

```cs
transaction.Sign(bitcoinPrivateKey, false);
```  

### Propagate your transactions
Congratulations, you have signed your first transaction! Your transaction is ready to roll! All that is left is to propagate it to the network so the miners can see it.  
#### With QBitNinja:  
```cs
BroadcastResponse broadcastResponse = client.Broadcast(transaction).Result;

if (!broadcastResponse.Success)
{
    Console.WriteLine("ErrorCode: " + broadcastResponse.Error.ErrorCode);
    Console.WriteLine("Error message: " + broadcastResponse.Error.Reason);
}
else
{
    Console.WriteLine("Success! You can check out the hash of the transaciton in any block explorer:");
    Console.WriteLine(transaction.GetHash());
}
```  

#### With your own Bitcoin Core:  

```cs
using (var node = Node.ConnectToLocal(network)) //Connect to the node
{
    node.VersionHandshake(); //Say hello
                             //Advertize your transaction (send just the hash)
    node.SendMessage(new InvPayload(InventoryType.MSG_TX, transaction.GetHash()));
    //Send it
    node.SendMessage(new TxPayload(transaction));
    Thread.Sleep(500); //Wait a bit
}
```  

The **using** code block will take care of closing the connection to the node. That's it!

You can also connect directly to the Bitcoin network, however I advise you to connect to your own trusted node (faster and easier)  