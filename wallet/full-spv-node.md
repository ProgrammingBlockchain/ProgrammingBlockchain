# Full SPV 노드 {#full-spv-node}

또 다른 더 새로운 비트코인 지갑 아키텍처로는 "Full Block Downloading SPV" 지갑이 있습니다.

이러한 지갑은 [Jonas Schnelli's Bitcoin Core PR](https://github.com/bitcoin/bitcoin/pull/9483), nopara73의 C# 으로 쓰여진 [HiddenWallet](https://github.com/nopara73/HiddenWallet) 및 [Stratis's Breeze Wallet](https://github.com/stratisproject/Breeze)이 있습니다.

full node와 비교하면, full SPV node는 모든 블록을 다운로드 하지만 지갑의 처음부터 다운로드 됩니다. 이것은 더 빠르지만 블록을 완전히 검증 할 수 없으며 대신 SPV 검증을 사용합니다.

주요 장점은 가벼운 지갑 아키텍처로 비교 됩니다: 네트워크 관찰자에 대한 full node 수준의 프라이버시를 제공합니다.