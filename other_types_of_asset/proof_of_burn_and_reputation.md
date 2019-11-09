## Proof of Burn and Reputation {#proof-of-burn-and-reputation}

The question is simple: in a P2P market were law enforcement is too expensive, how participants might minimize the probability to get scammed?

OpenBaazar seems [to be the first](https://gist.github.com/dionyziz/e3b296861175e0ebea4b) trying to use proof of burn as a reputation determinant.

There is several responses to that (escrow or notary/arbiter), but one that we will explore here is called Proof Of Burn.

Imagine yourself in the middle age, and you live in a small village with several local merchants.  
One day, a traveling merchant comes to your village and sells you some goods at an unbelievable low price compared to local one.

However, traveling merchant are well known for scamming people with low quality product, because losing reputation is a small price to pay for them compared to local merchants.  
Local Merchant invested into a nice store, advertising and their reputation. Unhappy customers can easily destroy them. But the traveling merchant, having no local store and only transient reputation do not have those incentives to not scam people.

On the internet, where the creation of an identity is so cheap, all merchants are potentially as the travelling one from the middle age.  
The solution of market providers was to gather the real identity of every participant in the market, so law enforcement become possible.

If you get scammed on Amazon or Ebay, your bank will most likely refund you, because they have a way to find the thief by contacting Amazon and Ebay.

In a purely P2P market using Bitcoin, we do not have that. If you get scam, you lose money.  
So how a buyer can trust the traveling merchant?  
The response is: by checking how much he invested into his reputation.

So as a good intentioned seller, you want to inspire confidence to your customer. For that you will destroy some of your wealth, and every customer will see. This is the definition of “investing into your reputation”.

Imagine you burned 50 BTC for your reputation. And a customer want to buy 2 BTC of goods from you. He has good reason to believe that you will not scam him, because you invested more into your reputation that what you can get out of him by scamming.  
It becomes not economically profitable for you to scam him.

The technical details will surely vary and change over time, but here is an example of Proof of Burn.  

```cs
var alice = new Key();

//Giving some money to alice
var init = Network.Main.CreateTransaction();
init.Outputs.Add(Money.Coins(1.0m), alice);

var coin = init.Outputs.AsCoins().First();

//Burning the coin
var burn = Network.Main.CreateTransaction();
burn.Inputs.Add(new TxIn(coin.Outpoint)
{
    ScriptSig = coin.ScriptPubKey
}); //Spend the previous coin

var message = "Burnt for \"Alice Bakery\"";
var opReturn = TxNullDataTemplate
                .Instance
                .GenerateScriptPubKey(Encoding.UTF8.GetBytes(message));
burn.Outputs.Add(Money.Coins(1.0m), opReturn);
burn.Sign(alice, false);

Console.WriteLine(burn);
```  

```json
{
  ….
  "in": [
    {
      "prev_out": {
        "hash": "0767b76406dbaa95cc12d8196196a9e476c81dd328a07b30954d8de256aa1e9f",
        "n": 0
      },
      "scriptSig": "304402202c6897714c69b3f794e730e94dd0110c4b15461e221324b5a78316f97c4dffab0220742c811d62e853dea433e97a4c0ca44e96a0358c9ef950387354fbc24b8964fb01 03fedc2f6458fef30c56cafd71c72a73a9ebfb2125299d8dc6447fdd12ee55a52c"
    }
  ],
  "out": [
    {
      "value": "1.00000000",
      "scriptPubKey": "OP_RETURN 4275726e7420666f722022416c6963652042616b65727922"
    }
  ]
}
```  

Once in the Blockchain, this transaction is undeniable proof that Alice invested money for her bakery.  
The Coin with ```ScriptPubKey OP_RETURN 4275726e7420666f722022416c6963652042616b65727922``` do not have any way to be spent, so those coins are lost forever.
