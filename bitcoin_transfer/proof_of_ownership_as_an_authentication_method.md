# Chapter8. Proof of an ownership as an authentication method {#proof-of-ownership-as-an-authentication-method}
> [Click to watch a video on YouTube [2016.05.02](https://www.youtube.com/watch?v=dZNtbAFnr-0)] My name is Craig Wright and I am about to demonstrate a signing of a message with the public key that is associated with the first transaction ever done in Bitcoin.  

```cs
var bitcoinSecretOfCraig = new BitcoinSecret("KzgjNRhcJ3HRjxVdFhv14BrYUKrYBzdoxQyR2iJBHG9SNGGgbmtC");
var message = "I am Craig Wright";
Console.WriteLine($"bitcoinSecretOfCraig.PrivateKey: {bitcoinSecretOfCraig.PrivateKey}");
string signature = bitcoinSecretOfCraig.PrivateKey.SignMessage(message);
Console.WriteLine($"signature: {signature}");
//Output:
//IN5v9+3HGW1q71OqQ1boSZTm0/DCiMpI8E4JB1nD67TCbIVMRk/e3KrTT9GvOuu3NGN0w8R2lWOV2cxnBp+Of8c=
Console.WriteLine($"bitcoinPrivateKey.Network: {bitcoinSecretOfCraig.Network}");
```  

Was that so hard?  

You may remember Craig Wright who really wanted us to believe he is Satoshi Nakamoto.  
He had successfully convinced a handful of influential Bitcoin people and journalists with some social engineering.  
Fortunately, digital signatures do not work that way.  
Let's quickly find the first ever bitcoin address associated with the genesis block, on the [Internet](https://en.bitcoin.it/wiki/Genesis_block):  
1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa
And verify his claim by the following code.  

```cs
var messageWrittenByCraig = "I am Craig Wright";
var signatureByCraig = "IN5v9+3HGW1q71OqQ1boSZTm0/DCiMpI8E4JB1nD67TCbIVMRk/e3KrTT9GvOuu3NGN0w8R2lWOV2cxnBp+Of8c=";
var bitcoinAddressOfTheFirstEver = new BitcoinPubKeyAddress("1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa");
//Verify the message and signature written by Craig by comparing them to ones located in the bitcoin address of the first ever.
bool isCraigWrightSatoshi = bitcoinAddressOfTheFirstEver.VerifyMessage(messageWrittenByCraig, signatureByCraig);
Console.WriteLine($"isCraigWrightSatoshi: {isCraigWrightSatoshi}");
```  

SPOILER ALERT! The bool will be false.  

It is how you prove you are the owner of the Bitcoin address by using message and signature without moving coins.  


**Exercise:** Verify that Nicolas sensei is not lying!

**Address:**  
[1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB](https://blockchain.info/address/1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB)  
**Message:**  
Nicolas Dorier Book Funding Address  
**Signature:**  
H1jiXPzun3rXi0N9v9R5fAWrfEae9WPmlL5DJBj1eTStSvpKdRR8Io6/uT9tGH/3OnzG6ym5yytuWoA9ahkC3dQ=  

This constitutes proof that Nicolas Dorier owns the private key of the book.  
  

## Sidenote
Do you know how PGP(Pretty Good Privacy) works? Pretty similar, right?  
Maybe this can be the foundation of a more user friendly PGP alternative.  
Please build it on top of the NBitcoin :-)
