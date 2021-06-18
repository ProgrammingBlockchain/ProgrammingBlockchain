## 블록체인 {#blockchain}

You might have noticed that while we proved ownership of the spent TxOut, we have not yet proven the TxOut actually exists. This is where the main function of the Blockchain shines:

The Blockchain is the database of all transactions that have happened since the the first Bitcoin transaction, known as the Genesis block. The Blockchain is duplicated all around the world. If you use Bitcoin Core, you have the whole Blockchain on your computer. Once a transaction appears on the Blockchain, it is very easy to prove its existence.

**Miners** are entities whose only goal is to insert a transaction in The Blockchain. However miners do not modify the blockchain everytime they receive one transaction. Instead each of them try to add a whole batch of transactions at the same time in something known as a **block**. Other nodes on the network confirm the new block obeys the rules set forth in the Bitcoin protocol. If two miners add a block at the same time, we have a **fork**, but ultimately only the branch of the fork with the most **work** will be continued. If a miner tries to include an invalid transaction in his block, the other nodes will not recognize it and the miner loses the investment spent on creating the block.

Once a miner manages to submit a valid block, all transactions inside are considered **Confirmed**. When this happens all miners must discard their current work and begin working on a new block using new transactions. When a block is confirmed it is added to the Blockchain as the most recent block. The likelihood of this addition being undone decreases dramatically with every subsequent block that is added on top of it.

For the first time in history we have a database which can’t easily be rewritten, eliminates the need for trust, resists censorship, and is widely distributed. Comparing the Blockchain to a ledger is only relevant if we consider Bitcoin as a currency.

The Blockchain is a database, and you give meaning to its data. As you will soon discover, a bitcoin transaction can bear more information than just bitcoin transfers. A bitcoin transaction is a row in a database that can never be erased.

As a user, you can verify that a specific transaction exists in the Blockchain in two different ways:

*   Check the entire Blockchain, which at the time of this writing is several gigabytes in size.
*   Ask for a partial Merkle tree, which are a few kilobytes in size. We will talk about Merkle trees later in relation to Simple Payment Verification (SPV).


사용 된 TxOut의 소유권을 증명했지만 TxOut이 실제로 존재한다는 사실을 아직 증명하지 않은 것을 눈치 챘을 것입니다. 이것이 블록체인의 주요 기능 입니다:

블록체인은 Genesis 블록으로 알려진 첫 비트코인 거래 이후 발생한 모든 거래의 데이터베이스입니다. 블록체인은 전 세계적으로 복제됩니다. Bitcoin Core를 사용하는 경우, 당신의 컴퓨터에 전체 블록체인이 저장 되어 있습니다. 거래가 블록체인에 한번 나타나면, 그 존재를 증명하는 것은 매우 쉽습니다.

**채굴자(Miners)**의 유일한 목표는 Blockchain에 자신의 거래를 추가하는 것 입니다. 그러나 채굴자는 하나의 트랜잭션을 받을 때마다 블록체인을 수정하지는 않습니다. 대신 그들 각각은 **block**으로 알려진 무언가를 동시에 전체 트랜잭션에 배치로 추가하게 됩니다. 네트워크의 또 다른 노드들은 새 블록이 비트코인 프로토콜에 명시된 규칙을 준수하는지 확인합니다. 두 명의 채굴자가 동시에 블록을 추가하게 되면 **fork**가 생기지만, 분기가 추가 된 이후 궁극적으로 가장 많은 **work**을 수행 한 포크의 분기만 생존 하게 됩니다. 한 채굴자가 자신의 블록에 유효하지 않은 트랜잭션을 포함하려고 하면, 다른 노드들이 이를 승인하지 않기 때문에, 채굴자는 블록 생성에 소비된 투자를 잃게 됩니다.

채굴자가 유효한 블록을 제출하면 내부의 모든 거래가 **Confirmed** 된 것으로 간주됩니다. 이런 일이 발생하면 모든 채굴자는 현재의 작업을 버리고 새 트랜잭션을 사용하여 새 블록에서 작업을 시작해야 합니다. 블록이 확인(confirmed)되면 가장 최근 블록으로 블록체인에 추가됩니다. 이 추가가 취소 될 가능성은 계속 추가되는 후속 블록이 늘어나면 극적으로 감소합니다.

우리는 역사상 처음으로 재작성 될 수 없고, 신뢰 문제가 해결되고, 중앙정부의 검열에 저항하고, 널리 배포되는 데이터베이스를 갖게 되었습니다. 블록체인을 원장과 비교하는 것은 비트코인을 통화로 간주하는 경우에만 관련이 있습니다.

블록체인은 데이터베이스이며, 데이터에 의미가 부여 됩니다. 곧 알게 되겠지만, 비트코인 거래는 단순한 비트코인 전송보다 더 많은 정보를 가질 수 있습니다. 비트코인 거래는 지울 수 없는 데이터베이스의 행(row) 입니다.

사용자는, 두 가지 방법으로 특정 트랜잭션이 블록체인에 존재하는지 확인할 수 있습니다.

* 이 글을 쓰는 시점에서 크기가 몇 기가 바이트인 전체 블록체인을 확인하십시오.
* 크기가 몇 킬로바이트인 부분 머클 트리를 요청하십시오. 나중에 SPV (Simple Payment Verification)와 관련하여 Merkle 트리에 대해 설명하겠습니다.