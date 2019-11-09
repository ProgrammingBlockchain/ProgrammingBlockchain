## Proof of ownership as an authentication method {#proof-of-ownership-as-an-authentication-method}
> [[2016.05.02](https://www.youtube.com/watch?v=dZNtbAFnr-0)] My name is Craig Wright and I am about to demonstrate a signing of a message with the public key that is associated with the first transaction ever done in Bitcoin.  

```cs
var bitcoinPrivateKey = new BitcoinSecret("XXXXXXXXXXXXXXXXXXXXXXXXXX", Network.Main);

var message = "I am Craig Wright";
string signature = bitcoinPrivateKey.PrivateKey.SignMessage(message);
Console.WriteLine(signature); // IN5v9+3HGW1q71OqQ1boSZTm0/DCiMpI8E4JB1nD67TCbIVMRk/e3KrTT9GvOuu3NGN0w8R2lWOV2cxnBp+Of8c=
```  

Was that so hard?  

You may remember Craig Wright, who really wanted us to believe he is Satoshi Nakamoto.  
He had successfully convinced a handful of influential Bitcoin people and journalists with some social engineering.  
Fortunately digital signatures do not work that way.  
Let's quickly find on the [Internet](https://en.bitcoin.it/wiki/Genesis_block) the first ever bitcoin address, associated with the genesis block: [1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa](https://blockchain.info/address/1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa) and verify his claim:  

```cs
var message = "I am Craig Wright";
var signature = "IN5v9+3HGW1q71OqQ1boSZTm0/DCiMpI8E4JB1nD67TCbIVMRk/e3KrTT9GvOuu3NGN0w8R2lWOV2cxnBp+Of8c=";

var address = new BitcoinPubKeyAddress("1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa", Network.Main);
bool isCraigWrightSatoshi = address.VerifyMessage(message, signature);

Console.WriteLine("Is Craig Wright Satoshi? " + isCraigWrightSatoshi);
```  

SPOILER ALERT! The bool will be false.   

Here is how you prove you are the owner of an address without moving coins:  

**Address:**  
[1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB](https://blockchain.info/address/1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB)  
**Message:**  
Nicolas Dorier Book Funding Address  
**Signature:**  
H1jiXPzun3rXi0N9v9R5fAWrfEae9WPmlL5DJBj1eTStSvpKdRR8Io6/uT9tGH/3OnzG6ym5yytuWoA9ahkC3dQ=  

This constitutes proof that Nicolas Dorier owns the private key of the book.  
**Exercise:** Verify that Nicolas sensei is not lying!  

### Sidenote
Do you know how PGP works? Pretty similar, right?  
Maybe this can be the foundation of a more user friendly PGP alternative.  
Please build it on top of NBitcoin :-)
