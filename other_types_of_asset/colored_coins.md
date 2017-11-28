## Chapter1. Colored Coins {#colored-coins}

So until now, you have seen how to exchange Bitcoins on the network. However, you can use the Bitcoin network for transferring and exchanging any type of assets.  

We call such assets “colored coins”.  
As far as the Blockchain is concerned, there is no difference between a Coin and a Colored Coin.

A colored coin is represented by a standard **TxOut**. Most of the time, such **TxOut** has a residual Bitcoin value called “Dust” (600 satoshi). And for a reference, "Dust" in the Blockchain system means the amount of coin value which is smaller than the transaction fee.

The real value of a colored coin resides in what the **issuer** of the coin will exchange against it.  

![](../assets/ColoredCoin.png)  

Since a colored coin is nothing but a standard coin with a special meaning, it follows all rules and protocols what you saw about proof of ownership and the feature of the **TransactionBuilder** also stays true. You can transfer a colored coin by exactly the same rules as we did before.  

As far as the blockchain is concerned, a **Colored Coin** is a **Coin** like all others.

You can represent several types of asset by a colored coin: company shares, bonds, stocks, and votes etc.

But no matter what types of asset you will represent, there will always have a trust relationship between the **issuer** of the asset and the **owner**.  
If you own some company share, then the company might decide to not send you dividends.  
If you own a bond, then the bank might not exchange it at maturity.

However, a violation of contract might be automatically detected with the help of **Ricardian Contracts**.  
A **Ricardian Contract** is a contract signed by the issuer with the rights attached to the asset. Such contract can be either human readable (PDF) or structured (JSON), so that tools can automatically prove any violation.  

The **issuer** can’t change the **ricardian contract** attached to an asset.

The Blockchain is only the transport medium of a financial instrument.  
The innovation is that everyone can create and transfer its own asset without intermediary, whereas traditional asset transport medium (clearing houses), are either heavily regulated, or purposefully kept secret, and closed to the general public.

**Open Asset** is the name of the protocol created by Flavien Charlon who describes how to **transfer** and **emit** colored coins on the Blockchain.  
Other protocols also exist, but Open Asset is the most easy and flexible and the only one supported by **NBitcoin**.

In the rest of the book, I will not go in the details of the Open Asset protocol. The GitHub page of the specification for Open Asset protocol would be better suited to this need.
