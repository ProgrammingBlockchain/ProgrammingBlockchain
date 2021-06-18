## Transaction {#transaction}

> ([Mastering Bitcoin](https://github.com/bitcoinbook/bitcoinbook/)) 트랜잭션은 비트코인 시스템에서 가장 중요한 부분입니다. 비트코인의 나머지 기능은 거래가 생성되고, 네트워크에서 전파되고, 검증되고, 최종적으로 글로벌 거래원장 (블록체인)에 추가 되도록 설계되었습니다. 트랜잭션은 비트코인 시스템 참가자간에 가치를 이전 하기위해 인코드 된 데이터 구조입니다. 각각의 거래는 복식부기 원장인 블록체인에 공개 기록 됩니다.


한개의 트랜잭션은 수신자가 없거나 여러명의 수신자를 가지게 됩니다. **발신자도 마찬가지입니다!** 블록체인에서, 발신자와 수신자는 이전 장에서 설명한 것처럼 항상 ScriptPubKey로 추상화 됩니다.

Bitcoin Core를 사용하는 경우 거래 탭에 다음과 같은 거래가 표시됩니다:


![](../assets/BitcoinCoreTransaction.png)  

**Transaction ID**를 주목하세요. 이 경우 거래ID는 ```f13dc48fb035bbf0a6e989a26b3ecb57b84f85e0836e777d6edf60d87a4a2d94``` 입니다.

> **Note:** 트랜잭션 ID는 "SHA256(SHA256(txbytes))"로 정의 할 수 있습니다.

> **Note:** 확인되지 않은 거래(unconfirmed)를 처리하기 위해 Transaction ID를 사용하지 마십시오. Transaction ID는 확인(confirmed) 되기 전에 조작 할 수 있습니다. 이를 "거래 가단성"이라고 합니다.


Blockchain.info 같은 블록 탐색기를 이용하면 트랜잭션을 확인 할 수 있습니다: https://blockchain.info/tx/f13dc48fb035bbf0a6e989a26b3ecb57b84f85e0836e777d6edf60d87a4a2d94
그러나 개발자는 더 쉽게 쿼리를 실행하거나 구문을 분석 할 수 있는 서비스를 원할 것입니다.

C# 개발자이자 NBitcoin 사용자 인 Nicolas Dorier의 [QBit Ninja](http://docs.qbitninja.apiary.io/)가 최적의 선택이라고 생각합니다. 블록체인을 조회하고 지갑을 추적하기 위한 오픈 소스 웹 서비스 API입니다.

QBit Ninja는 Microsoft Azure Storage를 기반으로 하는 [NBitcoin.Indexer](https://github.com/MetacoSA/NBitcoin.Indexer)에 의존하고 있습니다. C# 개발자는 이 API의 래퍼를 직접 개발하는 대신 클라이언트 라이브러리인 [NuGet client package](http://www.nuget.org/packages/QBitninja.Client)를 사용하는 것을 추천 합니다.

http://api.qbit.ninja/transactions/f13dc48fb035bbf0a6e989a26b3ecb57b84f85e0836e777d6edf60d87a4a2d94에 액세스 해 보면 트랜잭션의 내용을 볼 수 있습니다.


![](../assets/RawTx.png)  

다음 코드를 사용하면 16 진수(hex) 표현의 트랜잭션을 해석 할 수 있습니다:

```cs
Transaction tx = Transaction.Parse("0100000...", Network.Main);
```

두려워 지기전에 탭을 닫고, 다음 진행을 위해 QBit Ninja가 API를 쿼리하고 정보를 구문 분석 하도록 만든 **QBitNinja.Client** NuGet 패키지를 설치합니다.

![](../assets/QBitNuGet.png)  

다음과 같이 사용 합니다.

```cs
using QBitNinja.Client;
using QBitNinja.Client.Models;
```

id를 이용 트랙젝션을 쿼리 합니다:

```cs
// Create a client
QBitNinjaClient client = new QBitNinjaClient(Network.Main);
// Parse transaction id to NBitcoin.uint256 so the client can eat it
var transactionId = uint256.Parse("f13dc48fb035bbf0a6e989a26b3ecb57b84f85e0836e777d6edf60d87a4a2d94");
// Query the transaction
GetTransactionResponse transactionResponse = client.GetTransaction(transactionId).Result;
```  

**transactionResponse**의 형식은 **GetTransactionResponse** 입니다. QBitNinja.Client.Models 네임스페이스에 있습니다. **transactionResponse**로 **NBitcoin.Transaction** 형식을 얻을 수 있습니다:

```cs
NBitcoin.Transaction transaction = transactionResponse.Transaction;
```  
 
두 클래스를 사용하여 트랜잭션 ID를 반환하는 예를 살펴 보겠습니다:

```cs
Console.WriteLine(transactionResponse.TransactionId); // f13dc48fb035bbf0a6e989a26b3ecb57b84f85e0836e777d6edf60d87a4a2d94
Console.WriteLine(transaction.GetHash()); // f13dc48fb035bbf0a6e989a26b3ecb57b84f85e0836e777d6edf60d87a4a2d94
```  

**GetTransactionResponse**에는 트랜잭션에 사용되는 입력 값 및 scriptPubKey와 같은 트랜잭션에 대한 추가 정보가 있습니다.

현재 관련 부분은 **입력(inputs)** 과 **출력(outputs)** 입니다.

우리 트랜잭션에는 하나의 출력 만 있음을 알 수 있습니다. `13.19683492` 비트코인은 해당 ScriptPubKey로 전송됩니다.


```cs
List<ICoin> receivedCoins = transactionResponse.ReceivedCoins;
foreach (var coin in receivedCoins)
{
    Money amount = (Money) coin.Amount;

    Console.WriteLine(amount.ToDecimal(MoneyUnit.BTC));
    var paymentScript = coin.TxOut.ScriptPubKey;
    Console.WriteLine(paymentScript);  // It's the ScriptPubKey
    var address = paymentScript.GetDestinationAddress(Network.Main);
    Console.WriteLine(address); // 1HfbwN6Lvma9eDsv7mdwp529tgiyfNr7jc
    Console.WriteLine();
}
```  

QBitNinja의 GetTransactionResponse 클래스를 사용하여 RECEIVED COINS에 대한 정보를 작성했습니다. 

**Exercise**: QBitNinja의 GetTransactionResponse 클래스를 사용하여 SPENT COINS에 대한 동일한 정보를 작성하십시오!

NBitcoin의 Transaction 클래스를 사용하여 RECEIVED COINS에 대해 동일한 정보를 얻는 방법을 살펴 보겠습니다.

```cs
var outputs = transaction.Outputs;
foreach (TxOut output in outputs)
{
    Money amount = output.Value;

    Console.WriteLine(amount.ToDecimal(MoneyUnit.BTC));
    var paymentScript = output.ScriptPubKey;
    Console.WriteLine(paymentScript);  // It's the ScriptPubKey
    var address = paymentScript.GetDestinationAddress(Network.Main);
    Console.WriteLine(address);
    Console.WriteLine();
}
```  

이제 **inputs**을 살펴 보겠습니다. 그것을 들여다 보면 이전 출력이 참조 된 것을 알 수 있습니다. 각 입력들은 거래에 자금을 사용하기 위해 어떤 출력을 사용하고 있는지를 보여줍니다.

```cs
var inputs = transaction.Inputs;
foreach (TxIn input in inputs)
{
    OutPoint previousOutpoint = input.PrevOut;
    Console.WriteLine(previousOutpoint.Hash); // hash of prev tx
    Console.WriteLine(previousOutpoint.N); // idx of out from prev tx, that has been spent in the current tx
    Console.WriteLine();
}
```  

**TxOut**, **Output** 및 **out** 이라는 용어는 동의어입니다. 
**OutPoint**와 혼동하지 말아야 합니다, 나중에 좀더 자세히 설명하겠습니다.

요약하자면, TxOut는 비트코인의 양과 **ScriptPubKey**를 나타냅니다. (수신측)


![](../assets/TxOut.png)  

그림 처럼 현재 트랜잭션의 첫 번째 ScriptPubKey에서 21 비트코인으로 txout을 생성 해 보겠습니다:

```cs  
Money twentyOneBtc = new Money(21, MoneyUnit.BTC);
var scriptPubKey = transaction.Outputs[0].ScriptPubKey;
TxOut txOut = transaction.Outputs.CreateNewTxOut(twentyOneBtc, scriptPubKey);
```  

모든 **TxOut** 은 블록체인 수준에서 이를 포함하는 트랜잭션의 ID와 내부 인덱스에 의해 고유하게 처리됩니다. 이러한 참조를 **Outpoint**라고 합니다.

![](../assets/OutPoint.png)

예를 들어, 거래에서 13.19683492 BTC 가 있는 **TxOut**의 **Outpoint**는 (f13dc48fb035bbf0a6e989a26b3ecb57b84f85e0836e777d6edf60d87a4a2d94, 0)입니다.


```cs
OutPoint firstOutPoint = receivedCoins[0].Outpoint;
Console.WriteLine(firstOutPoint.Hash); // f13dc48fb035bbf0a6e989a26b3ecb57b84f85e0836e777d6edf60d87a4a2d94
Console.WriteLine(firstOutPoint.N); // 0
```  

이제 트랜잭션의 입력 (일명 **TxIn**)을 자세히 살펴 보겠습니다:

![](../assets/TxIn.png)

**TxIn**은 **TxOut**의 **Outpoint**와 또는, 소비 될 **ScriptSig**로 구성됩니다 (ScriptSig를 "소유권 증명(Proof of Ownership)"으로 볼 수 있음). 우리 거래에는 실제로 9 개의 입력이 있습니다.

```cs
Console.WriteLine(transaction.Inputs.Count); // 9
```  

이전 outpoint의 트랜잭션 ID를 사용하여 해당 트랜잭션과 관련된 정보를 검토 할 수 있습니다.


```cs
OutPoint firstPreviousOutPoint = transaction.Inputs[0].PrevOut;
var firstPreviousTransaction = client.GetTransaction(firstPreviousOutPoint.Hash).Result.Transaction;
Console.WriteLine(firstPreviousTransaction.IsCoinBase); // False
```  

우리는 채굴자가 새로 채굴 한 코인을 포함하는 **coinbase transaction**에 도달 할 때까지 이러한 방식으로 트랜잭션 ID를 계속 추적 할 수 있습니다.

**Exercise:** 코인베이스 거래를 찾을 때 까지 해당 거래와 그 조상의 첫 번째 입력을 따르십시오. 

Hint: 많은 단계가 있으므로 1~2 분 정도 걸릴 수 있지만 인내심을 가지세요! 

네, 맞습니다. 가장 효율적인 방법은 아니지만 좋은 연습입니다.

이 예에서 출력은 총 13.19683492 BTC입니다.

```cs
Money spentAmount = Money.Zero;
var spentCoins = transactionResponse.SpentCoins;

foreach (var spentCoin in spentCoins)
{
    spentAmount = (Money)spentCoin.Amount.Add(spentAmount);
}
Console.WriteLine(spentAmount.ToDecimal(MoneyUnit.BTC)); // 13.19703492
```  

이 거래에서는 13.19703492 BTC가 수신되었습니다.

**Exercise:** 지출 금액으로 한 것 처럼 총 수령 금액을 가져옵니다.

이는 0.0002 BTC (또는 13.19703492 - 13.19683492)가 계산되지 않음을 의미합니다! 입력과 출력의 차이를 **거래 수수료(Transaction Fees)** 또는 **채굴 수수료(Miner’s Fees)** 라고 합니다. 이것은 채굴자가 수집한 거래를 주어진 블록에 포함킨 money(코인) 입니다.


```cs
var fee = transaction.GetFee(spentCoins.ToArray());
Console.WriteLine(fee);
```

**coinbase transaction**은 총 output이 전체 input보다 큰 유일한 트랜잭션입니다. 이것은 코인 생성을 위한 효과적인 방법입니다. 따라서 정의에 따라 코인베이스 거래에는 수수료가 없습니다. 
코인베이스 트랜잭션은 모든 블록의 첫 번째 트랜잭션입니다. 합의 규칙은 코인베이스 거래에서 산출물 가치의 합계가 채굴 보상 (보조금과 블록의 거래 수수료 합계)을 초과하지 않도록 강제합니다.
