# SPV 노드 {#spv-node}

분산형 경량 클라이언트를 활성화하기 위해 SPV 지갑이 발명되었습니다. 
full SPV 지갑과 마찬가지로 SPV 지갑은 SPV 유효성 검사를 수행하지만 SPV 지갑은 비트 코인 노드에서 전체 블록이 아닌 트랜잭션만 다운로드합니다. 
이와 같이 단순한 클라이언트는 명백한 개인정보보호 단점으로 인해 구현되지 않았습니다. 
이러한 프라이버시 문제를 해결하기 위해 [BIP37](https://github.com/bitcoin/bips/blob/master/bip-0037.mediawiki)에 Bloom filtering SPV 지갑이 도입되어 인기를 얻었습니다. 나중에 Bloom filtering SPV 지갑 [turned out to be](https://jonasnick.github.io/blog/2015/02/12/privacy-in-bitcoinj/)은 개인정보보호의 악몽이 되었습니다.
개인정보보호가 다소 개선 된 C# 구현 샘플은 GitHub: [NBitcoin.SPVSample](https://github.com/NicolasDorier/NBitcoin.SPVSample)에서 찾을 수 있습니다.

Compact Client Side Filtering이라고 하는 최신 버전의 SPV 지갑은 다른 아키텍처를 사용하여 BIP37의 개인 정보를 향상시킵니다. [Neutrino](https://github.com/lightninglabs/neutrino)라는 이러한 구현이 현재 존재합니다.
