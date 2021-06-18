## 비트코인 주소 {#bitcoin-address}

**비트코인 주소**는 다른 사람이 당신에게 코인을 송금 할 수 있도록 세상에 공개 하는 주소 입니다.

![](../assets/BitcoinAddress.png)  

공개 주소로 송금 받은 코인을 사용하려면 **개인 키(private-key)** 가 필요 합니다.

![](../assets/PrivateKey.png)  

개인 키는 네트워크에 저장되지 않으며 인터넷에 연결하지 않고도 생성 할 수 있습니다.

아래 코드는 NBitcoin으로 개인 키를 생성하는 방법은 다음과 같습니다.

```cs  
Key privateKey = new Key(); // generate a random private key
```  

개인 키로 단방향 암호화 기능을 이용하여 **공개 키(public-key)** 를 생성 가능합니다 

![](../assets/PrivKeyPubKey.png)  

```cs 
PubKey publicKey = privateKey.PubKey;
Console.WriteLine(publicKey); // 0251036303164f6c458e9f7abecb4e55e5ce9ec2b2f1d06d633c9653a07976560c
```  

비트코인은 두 가지의 **network**이 있습니다: 

* **TestNet**은 개발 목적의 비트코인 네트워크 입니다. 이 네트워크의 비트코인은 가치가 없습니다.  
* **MainNet**은 모두가 사용하는 비트코인 네트워크 입니다. 

> **메모:** TestNet에서 사용되는 코인은 **faucets**에서 얻을 수 있습니다. "get testnet bitcoins"으로 구글링 하세요.

비트코인 **network**에서 공개 키를 사용하여 **비트코인 주소**를 쉽게 얻을 수 있습니다.

![](../assets/PubKeyToAddr.png)  

```cs 
Console.WriteLine(publicKey.GetAddress(ScriptPubKeyType.Legacy, Network.Main)); // 1PUYsjwfNmX64wS368ZR5FMouTtUmvtmTY
Console.WriteLine(publicKey.GetAddress(ScriptPubKeyType.Legacy, Network.TestNet)); // n3zWAo2eBnxLr3ueohXnuAa8mTVBhxmPhq
```  

**정확하게 표현 하자면, 비트코인 주소는 버전을 나타내는 선두 1 바이트 (TestNet 또는 MainNet을 구분하는 1 바이트)와 공개 키 해시로 구성되어 있으며, 그들이 결합 된 후 Base58Check 인코딩 됩니다.**

![](../assets/PubKeyHashToBitcoinAddress.png)  

```cs 
var publicKeyHash = publicKey.Hash;
Console.WriteLine(publicKeyHash); // f6889b21b5540353a29ed18c45ea0031280c42cf
var mainNetAddress = publicKeyHash.GetAddress(Network.Main);
var testNetAddress = publicKeyHash.GetAddress(Network.TestNet);
```  

> **Fact:** 공개 키를 SHA256 해시 한 결과에 Big Endian 표기법을 사용하여 RIPEMD160 해시 처리 합니다. 코딩 방법은 다음과 같습니다: RIPEMD160(SHA256(pubkey))  

Base58Check 인코딩에는 오타를 방지하기 위한 체크섬 기능과 '0'(Zero) 및 'O'(alphabet)와 같은 모호한 문자의 사용을 방지하는 기능이 있습니다.
Base58Check 인코딩 된 비트코인 주소는 지갑 사용자에게 네트워크를 결정하는 일관된 방법을 제공합니다. 지갑이 MainNet 코인을 TestNet 주소로 보내는 것을 방지합니다.

```cs 
Console.WriteLine(mainNetAddress); // 1PUYsjwfNmX64wS368ZR5FMouTtUmvtmTY
Console.WriteLine(testNetAddress); // n3zWAo2eBnxLr3ueohXnuAa8mTVBhxmPhq
```  

> **Tip:** 메인넷(MainNet)에서 비트코인 프로그래밍을 연습하는 경우 문제 발생시 복구 불가능 합니다. 연습은 테스트넷(TestNet)을 이용하세요.