# Full Node {#full-node}

가장 오래되고 가장 권장되는 솔루션입니다.

NBitcoin 통해 Stratis가 만든 [C# Bitcoin Full Node](https://github.com/stratisproject/StratisBitcoinFullNode)를 또는 [Bitcoin Core's RPC](https://bitcoin.org/en/developer-reference#remote-procedure-calls-rpcs)를 사용할 수 있습니다.

장점은 무신뢰성(trustlessness)과 높은 네트워크 수준의 개인정보보호이며, 단점은 고용량, 고대역폭, 높은 CPU 처리 능력 및 더 많은 시간이 요구됩니다.

이 문서의 나머지 부분에서는 Bitcoin Core의 RPC API에 대해 설명합니다. 

먼저 비트 코인 코어 노드를 완전히 동기화 한 다음 비트 코인 코어 인스턴스에서 RPC 명령을 호출하여 위의 모든 작업을 수행 할 수 있습니다.


![Bitcoin Core](../assets/Wallet-Bitcoin-Core.png)

며칠이 걸릴 수 있는 초기 블록 체인 다운로드 (IBD: Initial Blockchain Downloading) 후 `bitcoind`을 시작합니다. 그런 다음 NBitcoin의 `RPCClient` 클래스를 사용하여 지갑을 관리하게 됩니다.

```
RPCClient client = new RPCClient(Network.Main);
Console.WriteLine(client.GetNewAddress()); # Generate a new address
Console.WriteLine(client.GetBalance()); # Get the balance
```

장점:

* 광범위하게 테스트 된 소프트웨어,
* 논쟁의 여지가 있는 포크(fork)의 경우 어떤 포크(fork)를 따를 지 결정 할 수 있습니다.
* 휼륭한 문서

단점:

* API는 때때로 사용하기 어렵고 번거로움,
* 완전히 동기화된 노드를 받기 위한 IBD(Initial Blockchain Downloading)만 며칠이 걸립니다.
* 비트 코인 코어는 많은 트랜잭션 처리를 해야하는 지갑에는 충분하지 않을 수 있습니다.
* 지갑 추가를 위해 `bitcoind` 재시작 필요
* 제한된 수의 지갑을 지원합니다. (10 개 미만?)


거래량이 적고 시스템에서 지갑을 동적으로 생성 할 필요가 없는 경우 이 솔루션을 권장합니다.

비트 코인 코어의 이전 버전은 지갑 계정을 지원했습니다. 

더 이상 그렇지는 않지만 RPC 위에 직접 계정을 구현하거나, 여전히 계정 기능을 유지하고 있는 luke-jr의 Bitcoin Core 포크(fork) 인 [Bitcoin Knots](https://github.com/bitcoinknots/bitcoin)를 사용할 수 있습니다. 

