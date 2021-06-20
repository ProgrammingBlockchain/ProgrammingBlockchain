## 코인 사용 {#spend-your-coin}


**bitcoin address**,**ScriptPubKey**,**private key** 그리고 **miner**에 대해서 알게 되었고, 이제 직접 첫 번째 **transaction**을 만들게 될 것입니다.

이 레슨을 진행하면서 Twitter 스타일 메시지에 책에 대한 피드백을 남길 방법을 빌드하기 위해 제시된대로 코드를 한 줄씩 추가합니다.

먼저 TestNet에서 지침을 따르고 메인 비트코인 네트워크에서 다시 수행하는 것이 좋습니다.

이전과 같이 지출하려는 **TxOut**이 포함 된 **transaction**을 살펴 보겠습니다 .

새 **Console Project** (.net45)를 만들고 QBitNinja.Client NuGet을 설치 합니다.

이미 개인 키를 생성하고 기록해 두셨습니까? 해당 비트코인 주소를 이미 받고 거기에 자금을 보냈습니까? 그렇지 않다면 걱정하지 마십시오. 어떻게 할 수 있는지 빠르게 다시 알려 주겠습니다:


```cs
// Replace this with Network.Main to do this on Bitcoin MainNet
var network = Network.TestNet;

var privateKey = new Key();
var bitcoinPrivateKey = privateKey.GetWif(network);
var address = bitcoinPrivateKey.GetAddress();

Console.WriteLine(bitcoinPrivateKey);
Console.WriteLine(address);
```

먼저 TestNet을 사용하지만 MainNet에서도 이 작업을 수행하므로 실제 돈을 지출하게됩니다! 어떤 경우든, **bitcoinPrivateKey** 및 주소를 기록해 두세요! 거기에 몇 달러의 코인을 보내고 트랜잭션 ID를 저장하십시오 (지갑 소프트웨어 또는 SmartBit for [MainNet](http://smartbit.com.au/) 및 [TestNet](https://testnet.smartbit.com.au/) 과 같은 블록 탐색기에서 찾을 수 있음).

개인 키를 가져옵니다 ( "cN5Y ... K2RS" 문자열을 귀하의 것으로 대체):


```cs
var bitcoinPrivateKey = new BitcoinSecret("cN5YQMWV8y19ntovbsZSaeBxXaVPaK4n7vapp4V56CKx5LhrK2RS", Network.Testnet);
var network = bitcoinPrivateKey.Network;
var address = bitcoinPrivateKey.GetAddress();

Console.WriteLine(bitcoinPrivateKey); // cN5YQMWV8y19ntovbsZSaeBxXaVPaK4n7vapp4V56CKx5LhrK2RS
Console.WriteLine(address); // mkZzCmjAarnB31n5Ke6EZPbH64Cxexp3Jp
```

마지막으로 거래 정보를 얻습니다 ( "0acb ... b78a"를 코인을 보낸 후 지갑 소프트웨어 또는 블록체인 탐색기에서 얻은 정보로 대체):

```cs
var client = new QBitNinjaClient(network);
var transactionId = uint256.Parse("0acb6e97b228b838049ffbd528571c5e3edd003f0ca8ef61940166dc3081b78a");
var transactionResponse = client.GetTransaction(transactionId).Result;

Console.WriteLine(transactionResponse.TransactionId); // 0acb6e97b228b838049ffbd528571c5e3edd003f0ca8ef61940166dc3081b78a
Console.WriteLine(transactionResponse.Block.Confirmations); // 91
```

이제 우리는 거래를 생성하는 데 필요한 모든 정보를 얻었습니다. 주요 질문은 다음과 같습니다: **from where, to where and how much?**

### From where?

우리의 경우 두 번째 아웃 포인트를 사용하려고합니다. 우리가 이것을 알아 낸 방법은 다음과 같습니다:

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

지불을 위해 거래에서 이 아웃 포인트를 참조해야합니다. 다음과 같이 트랜잭션을 생성합니다:

```cs
var transaction = Transaction.Create(network);
transaction.Inputs.Add(new TxIn()
{
    PrevOut = outPointToSpend
});
```

### To where?

주요 질문을 기억하십니까? **From where, to where and how much?** 
**TxIn**을 구성하고 트랜잭션에 추가하는 것이 "from where" 질문에 대한 답입니다.
**TxOut**을 구성하고 트랜잭션에 추가하는 것이 나머지 항목에 대한 답입니다. 

> 이 책의 기부 주소 : [1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB](https://www.smartbit.com.au/address/1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB) 이 돈은 Nicolas의 "Coffee and Sushi Wallet"으로 입금되서 그가 나머지 책을 쓰는 동안 그를 지원하게 될 것입니다. MainNet에서 이 챌린지를 성공적으로 완료하면 [http://n.bitcoin.ninja/](http://n.bitcoin.ninja/) 상의 제작자의 전당 **Hall of the Makers**에서 기여한 내용을 찾을 수 있습니다 (기여 순서로 정렬).

메인 넷 주소 만들기:

```cs
var hallOfTheMakersAddress = new BitcoinPubKeyAddress("1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB", Network.Main);
```

또는 TestNet에서 작업하는 경우 TestNet 코인을 임의의 주소로 보내십시오. 나는 [mzp4No5cmCXjZUpf112B1XWsvWBfws5bbB](https://testnet.smartbit.com.au/address/mzp4No5cmCXjZUpf112B1XWsvWBfws5bbB)를 사용 했습니다.

```cs
var hallOfTheMakersAddress = BitcoinAddress.Create("mzp4No5cmCXjZUpf112B1XWsvWBfws5bbB", Network.TestNet);
```

### How much?

비트코인은 [사용할 수있는 여러 단위](https://en.bitcoin.it/wiki/Units)가 있지만, 알아야 할 세 가지가 있습니다 : 비트코인, 비트 및 사토시. 1 비트코인 (BTC)은 1,000,000 비트이고 100 사토시는 1 비트입니다. 1 satoshi (sat)는 비트코인 네트워크에서 가장 작은 단위입니다.

**0.001 BTC**를 보유하고 있는 경우, **사용되지 않은 출력(unspent)**에서 **0.0004 BTC** (a few dollars)를 보내려면, 실제로는 전부를 보내게 됩니다!
아래 다이어그램에서 볼 수 있듯이 **transaction output**은 [Hall of The Makers](http://n.bitcoin.ninja/)에 **0.0004 BTC**를 할당하고 **0.00053 BTC**는 반환됩니다.
나머지 **0.00007 BTC**는 어떻게 되나요? 이것은 채굴자에게 지급되는 수수료 입니다.
채굴자 수수료는 채굴자가 이 거래를 다음 블록에 추가하도록 합니다. 채굴 수수료가 높을수록 채굴자가 다음 블록에 거래를 포함시키려는 동기가 높아져 거래가 더 빨리 확인됩니다. 채굴 수수료를 0으로 설정하면 거래가 확인되지 않을 수 있습니다. 


![](../assets/SpendTx.png)

```cs
transaction.Outputs.Add(Money.Coins(0.0004m), hallOfTheMakersAddress.ScriptPubKey);
// Send the change back
transaction.Outputs.Add(new Money(0.00053m, MoneyUnit.BTC), bitcoinPrivateKey.ScriptPubKey);
```

여기서 미세 조정을 할 수 있습니다. 채굴 수수료를 기준으로 변경 사항을 계산해 보겠습니다. 

```cs
// How much you want to spend
var hallOfTheMakersAmount = new Money(0.0004m, MoneyUnit.BTC);

// How much miner fee you want to pay
/* Depending on the market price and
 * the currently advised mining fee,
 * you may consider to increase or decrease it.
 */
var minerFee = new Money(0.00007m, MoneyUnit.BTC);

// How much you want to get back as change
var txInAmount = (Money)receivedCoins[(int) outPointToSpend.N].Amount;
var changeAmount = txInAmount - hallOfTheMakersAmount - minerFee;
```

TxOut에 대해 계산 된 값을 사용하겠습니다:

```cs
transaction.Outputs.Add(hallOfTheMakersAmount, hallOfTheMakersAddress.ScriptPubKey);
// Send the change back
transaction.Outputs.Add(changeAmount, bitcoinPrivateKey.ScriptPubKey);
```

### Message on The Blockchain

개인 피드백을 추가 해 보세요! 80 바이트 이하여야 합니다. 그렇지 않으면 거래가 거부됩니다.
이 메시지는 transaction에 포함되어 거래가 확인 된 후 [Hall of The Makers](http://n.bitcoin.ninja/)에 표시 됩니다! :) 

```cs
var message = "Long live NBitcoin and its makers!";
var bytes = Encoding.UTF8.GetBytes(message);
transaction.Outputs.Add(Money.Zero, TxNullDataTemplate.Instance.GenerateScriptPubKey(bytes));
```

### Summary

마지막으로, 서명(signature)하기 전에 전체 거래를 살펴 보겠습니다:
3개의 **출력(TxOut)**, 2개의 값이 있는 **코인(value)**, 값이 없는 (메시지를 담고 있는) 1개의 **코인(value)**이 있습니다. 메시지 내에서 "일반적인" **출력(TxOut)**의 **scriptPubKey** 와 메시지가 있는 **출력(TxOut)**의 **scriptPubKey** 사이의 차이를 확인 할 수 있습니다.

```json
{
  "hash": "eeffd48b317e7afa626145dffc5a6e851f320aa8bb090b5cd78a9d2440245067",
  "ver": 1,
  "vin_sz": 1,
  "vout_sz": 3,
  "lock_time": 0,
  "size": 164,
  "in": [
    {
      "prev_out": {
        "hash": "0acb6e97b228b838049ffbd528571c5e3edd003f0ca8ef61940166dc3081b78a",
        "n": 0
      },
      "scriptSig": ""
    }
  ],
  "out": [
    {
      "value": "0.00040000",
      "scriptPubKey": "OP_DUP OP_HASH160 d3a689bc36464b9d74e1721fd321d4686eae594e OP_EQUALVERIFY OP_CHECKSIG"
    },
    {
      "value": "0.00053000",
      "scriptPubKey": "OP_DUP OP_HASH160 376b786582a3423bcdda4517ea87f0a7e862f27b OP_EQUALVERIFY OP_CHECKSIG"
    },
    {
      "value": "0.00000000",
      "scriptPubKey": "OP_RETURN 4c6f6e67206c697665204e426974636f696e20616e6420697473206d616b65727321"
    }
  ]
}
```

**TxIn**을 자세히 살펴보세요. **prev\_out** 및 **scriptSig**가 있습니다.
**Exercise:** 추가 정보를 읽기 전에 **scriptSig**가 무엇이고 어떻게 코드에 포함되는지 알아 보십시오!

TestNet blockexplorer에서 **prev\_out**의 **hash**를 확인해 봅시다: [prev\_out tx details](https://testnet.smartbit.com.au/tx/0acb6e97b228b838049ffbd528571c5e3edd003f0ca8ef61940166dc3081b78a). 0.001 BTC가 우리 주소로 전송되었음을 알 수 있습니다.

**prev\_out** **n**은 0입니다. 0에서 인덱싱하므로 트랜잭션의 첫 번째 출력을 사용한다는 의미입니다 (두 번째 출력은 트랜잭션의 1.0989548 BTC 입니다). 


### Sign your transaction


이제 트랜잭션을 생성 했으므로 서명해야합니다. 즉, 입력에서 참조한 TxOut을 소유하고 있음을 증명해야합니다.

서명은 [복잡](https://en.bitcoin.it/w/images/en/7/70/Bitcoin_OpCheckSig_InDetail.png) 할 수 있지만 간단하게 만들겠습니다.

먼저 **in**의 **scriptSig** 코드에서 어떻게 가져올 수 있는지 다시 살펴 보겠습니다. 주소의 ScriptPubKey로 ScriptSig를 채우는 것에는 두 가지 방법이 있습니다:


```cs
// Get it from the public address
var address = BitcoinAddress.Create("mkZzCmjAarnB31n5Ke6EZPbH64Cxexp3Jp", Network.TestNet);
transaction.Inputs[0].ScriptSig = address.ScriptPubKey;

// OR we can also use the private key 
var bitcoinPrivateKey = new BitcoinSecret("cN5YQMWV8y19ntovbsZSaeBxXaVPaK4n7vapp4V56CKx5LhrK2RS", Network.TestNet);
transaction.Inputs[0].ScriptSig =  bitcoinPrivateKey.ScriptPubKey;
```

transaction에 서명하려면 개인 키가 필요 합니다:

```cs
transaction.Sign(bitcoinPrivateKey, receivedCoins.ToArray());
```

이 명령 후에 입력의 ScriptSig 속성이 서명으로 대체되고, 트랜잭션이 서명 완료 됩니다.

블록체인 탐색기 [여기](https://testnet.smartbit.com.au/tx/eeffd48b317e7afa626145dffc5a6e851f320aa8bb090b5cd78a9d2440245067)에서 TestNet 트랜잭션을 확인할 수 있습니다. 


### Propagate your transactions (거래 전파)

축하합니다. 첫 거래에 서명하셨습니다! 거래를 시작할 준비가되었습니다! 남은 것은 채굴자가 볼 수 있도록 네트워크에 전파하는 것입니다. 

#### With QBitNinja:

```cs
BroadcastResponse broadcastResponse = client.Broadcast(transaction).Result;

if (!broadcastResponse.Success)
{
    Console.Error.WriteLine("ErrorCode: " + broadcastResponse.Error.ErrorCode);
    Console.Error.WriteLine("Error message: " + broadcastResponse.Error.Reason);
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

**using** 코드 블록은 노드에 대한 연결 종료를 보장 합니다.

비트코인 네트워크에 직접 연결할 수도 있지만, 더 빠르고 쉽게 신뢰할 수있는 노드에 연결하는 것이 좋습니다. 

## Need more practice?

YouTube: [How to make your first transaction with NBitcoin](https://www.youtube.com/watch?v=X4ZwRWIF49w)  
CodeProject: [Create a Bitcoin transaction by hand.](http://www.codeproject.com/Articles/1151054/Create-a-Bitcoin-transaction-by-hand)  
CodeProject: [Build your own Bitcoin wallet](https://www.codeproject.com/Articles/1115639/Build-your-own-Bitcoin-wallet)

