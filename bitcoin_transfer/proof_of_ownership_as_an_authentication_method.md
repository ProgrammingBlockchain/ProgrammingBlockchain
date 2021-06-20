## 인증 방법으로 소유권 증명 {#proof-of-ownership-as-an-authentication-method}
> [[2016.05.02](https://www.youtube.com/watch?v=dZNtbAFnr-0)] 내 이름은 크레이그 라이트이고 나는 비트코인에서 수행 된 첫 번째 거래와 관련된 공개 키와 메시지의 서명을 시연하려고합니다.

```cs
var bitcoinPrivateKey = new BitcoinSecret("XXXXXXXXXXXXXXXXXXXXXXXXXX", Network.Main);

var message = "I am Craig Wright";
string signature = bitcoinPrivateKey.PrivateKey.SignMessage(message);
Console.WriteLine(signature); // IN5v9+3HGW1q71OqQ1boSZTm0/DCiMpI8E4JB1nD67TCbIVMRk/e3KrTT9GvOuu3NGN0w8R2lWOV2cxnBp+Of8c=
```  

그렇게 힘들었나요?

나카 모토 사토시라고 믿기를 정말로 원했던 크레이그 라이트를 기억할 것입니다.

그는 소수의 영향력있는 비트코인 사람들과 언론인을 사회공학적인 방법으로 설득에 성공 했습니다.

다행히 디지털 서명은 그렇게 작동하지 않습니다. [Internet](https://en.bitcoin.it/wiki/Genesis_block)에서 제네시스 블록: [1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa](https://blockchain.info/address/1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa)와 관련된 최초의 비트코인 주소를 찾아보고 그의 주장을 확인 해 봅시다:

```cs
var message = "I am Craig Wright";
var signature = "IN5v9+3HGW1q71OqQ1boSZTm0/DCiMpI8E4JB1nD67TCbIVMRk/e3KrTT9GvOuu3NGN0w8R2lWOV2cxnBp+Of8c=";

var address = new BitcoinPubKeyAddress("1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa", Network.Main);
bool isCraigWrightSatoshi = address.VerifyMessage(message, signature);

Console.WriteLine("Is Craig Wright Satoshi? " + isCraigWrightSatoshi);
```  

스포일러 경고! 결과 값은 거짓입니다.

다음은 코인을 이동하지 않고 주소의 소유자임을 증명하는 방법입니다:

**Address:**  
[1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB](https://blockchain.info/address/1KF8kUVHK42XzgcmJF4Lxz4wcL5WDL97PB)  

**Message:**  
Nicolas Dorier Book Funding Address  

**Signature:**  
H1jiXPzun3rXi0N9v9R5fAWrfEae9WPmlL5DJBj1eTStSvpKdRR8Io6/uT9tGH/3OnzG6ym5yytuWoA9ahkC3dQ=  

이것은 Nicolas Dorier가 책의 개인 키를 소유하고 있다는 증거입니다.

**Exercise:** Nicolas sensei가 거짓말을하고 있지 않은지 확인하세요!

### Sidenote

PGP가 어떻게 작동하는지 아십니까? 꽤 비슷 하죠?

아마도 이것은보다 사용자 친화적 인 PGP 대안의 기초가 될 수 있습니다.

NBitcoin 위에 구축 해주세요 :-)