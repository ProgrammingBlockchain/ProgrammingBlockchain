## ScriptPubKey {#payment-script}

사실 비트코인 블록체인의 내부에는 비트코인 주소 같은 것은 없습니다. 내부적으로 비트코인의 프로토콜에서는 수신자를 **ScriptPubKey**로 식별 합니다.

![](../assets/ScriptPubKey.png)  

**ScriptPubKey**는 다음과 같은 형식으로 표현 됩니다:  

```OP_DUP OP_HASH160 14836dbe7f38c5ac3d49e8d790af808a4ee9edcf OP_EQUALVERIFY OP_CHECKSIG```  

비트코인의 소유권을 주장하기 위한 조건을 설명하는 짧은 스크립트입니다. 이 책의 강의를 진행 하면서 **ScriptPubKey** 의 작업 유형에 대해 살펴 보겠습니다 .

비트코인 주소로 ScriptPubKey를 생성 할 수 있습니다. 모든 비트코인 클라이언트가 "인간 친화적"인 비트코인 주소를 블록체인이 읽을 수 있는 주소로 변환하는 단계입니다.

![](../assets/BitcoinAddressToScriptPubKey.png)  

```cs 
var publicKeyHash = new KeyId("14836dbe7f38c5ac3d49e8d790af808a4ee9edcf");

var testNetAddress = publicKeyHash.GetAddress(Network.TestNet);
var mainNetAddress = publicKeyHash.GetAddress(Network.Main);

Console.WriteLine(mainNetAddress.ScriptPubKey); // OP_DUP OP_HASH160 14836dbe7f38c5ac3d49e8d790af808a4ee9edcf OP_EQUALVERIFY OP_CHECKSIG
Console.WriteLine(testNetAddress.ScriptPubKey); // OP_DUP OP_HASH160 14836dbe7f38c5ac3d49e8d790af808a4ee9edcf OP_EQUALVERIFY OP_CHECKSIG
```  

testnet과 mainnet 주소에 대한 **ScriptPubKey**가 동일할까요?

**ScriptPubKey**에는 공개키의 해시 값이 들어 있나요?

세부 사항은 아직 다루지 않겠지만, **ScriptPubKey**는 비트코인 주소와 관련이 없는 것 처럼 보이지만, 공개 키의 해시 값을 보여줍니다.

비트코인 주소는 네트워크를 가리키는 1 바이트 버전 값과 공개 키의 해시 값으로 구성 되어 있습니다. 그래서 **ScriptPubKey**와 네트워크 식별자를 이용해 비트코인 주소를 생성 할 수 있습니다.

```cs
var paymentScript = publicKeyHash.ScriptPubKey;
var sameMainNetAddress = paymentScript.GetDestinationAddress(Network.Main);
Console.WriteLine(mainNetAddress == sameMainNetAddress); // True
```   

**ScriptPubKey**로 부터 해시 값 및 비트코인 주소를 생성 할 수 있습니다.

```cs
var samePublicKeyHash = (KeyId) paymentScript.GetDestination();
Console.WriteLine(publicKeyHash == samePublicKeyHash); // True
var sameMainNetAddress2 = new BitcoinPubKeyAddress(samePublicKeyHash, Network.Main);
Console.WriteLine(mainNetAddress == sameMainNetAddress2); // True
```   

> **Note:** ScriptPubKey에는 비트코인 사용이 허용 된 해시 된 공개 키가 반드시 포함 될 필요는 없습니다.

이제 개인 키, 공개 키, 공개 키 해시, 비트코인 주소 및 ScriptPubKey 간의 관계를 이해했습니다.

이 책의 나머지 부분에서는 **ScriptPubKey** 만 사용하게 됩니다. 비트코인 주소는 단지 사용자와 상호 대화용 일뿐 입니다.
