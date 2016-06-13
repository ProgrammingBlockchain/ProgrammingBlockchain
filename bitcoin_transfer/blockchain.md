## Blockchain {#blockchain}

You might have noticed that while we proved ownership of the spent TxOut, and that we have not proven the TxOut actually exists. This is where the main function of the Blockchain shines:

The Blockchain is the database of all transactions that have happened since the the first Bitcoin transaction, known as the Genesis block. The Blockchain is duplicated all around the world. If you use Bitcoin Core, you have the whole Blockchain on your computer. Once a transaction appears on the Blockchain, it is very easy to prove its existence.

**Miners** are entities whose only goal is to insert a transaction in The Blockchain. However miners do not modify the blockchain everytime they receive one transaction. Instead each of them try to add a whole batch of transaction at the same time known as a **block**. Other nodes on the network confirm the new block obeys the rules set forth in the Bitcoin protocol. If two miners add a block at the same time, we have a **fork**, ultimately only the branch of the fork with the most **work** will be continued. If a miner tries to include an invalid transaction in his block, the other nodes will not recognize it and the miner loses the investment spent on creating the block.

Once a miner manages to submit a valid block all transactions inside are considered **Confirmed**. When this happens all miners must discard their current work and begin with new transactions. When a block is confirmed it is written into the Blockchain, and the likelihood of it being undone decreases dramatically with every subsequent block.

For the first time in history we have a database which can’t easily be rewritten, eliminates the need for trust, resists censorship, and is widely distributed. Comparing the Blockchain to a ledger is only relevant if we consider Bitcoin as a currency.

The Blockchain is a database, and you give meaning to its data. As you will soon discover, a bitcoin transaction can bear more information than just bitcoin transfers. A bitcoin transaction is a row in a database that can never be erased.

As a user, you can verify that a specific transaction exists in the Blockchain in two different ways:

*   Check the entire Blockchain, which at the time of this writing is several gigabytes in size.
*   Ask for a partial merkel tree, which are a few kilobytes. We will talk about merkel trees more as it relates to Simple Payment Verification.
