## Liquid Democracy {#liquid-democracy}

### Ikhtisar {#overview}

Pada pembahasan ini adalah murni latihan secara konseptual, salah satu pengaplikasian dari colored coins.

Mari kita bayangkan pada sebuah perusahaan yang mengambil beberapa keputusan oleh dewan investor setelah melakukan pemungutan suara.

* Beberapa investor mungkin tidak cukup mengetahui tentang sebuah topik, sehingga mereka ingin mendelegasikan keputusan tentang beberapa hal kepada orang lain. 
* Berpotensi pada sejumlah besar investor. 
* Sebagai seorang CEO, anda ingin mempunyai kemampuan untuk menjual hak suara untuk pembiayaan perusahaan. 
* Sebagai seorang CEO, anda ingin memberikan suara ketika anda harus memutuskan sesuatu. 

Bagaimana Colored Coins dapat membantu memberikan suara secara transparan?

Sebelum memulai, mari kita berbicara tentang beberapa _downside \(kekurangan\)_ dari sebuah voting dalam Blockchain:

* Tidak ada yang mengetahui identitas asli pemberi suara. 
* Penambang bisa menyensor \(bahkan jika itu akan bisa dibuktikan, tidak dalam kepentingan mereka\)
* Meski tidak ada yang mengetahui identitas asli pemberi voting, analisis perilaku pemilih pada beberapa orang mungkin bisa mengungkapkan identitasnya. 

Apakah poin-poin tersebut dianggap cukup relevan atau tidak, itu terserah penyelenggara pemungutan suara untuk memutuskannya.

Sekarang mari kita melihat gambaran bagaimana kita dapat melakukannya:

### Menerbitkan voting power {#issuing-voting-power}

Tentu saja, semua akan bermula dari keputusan pendiri perusahaan \(atau kita bisa menyebutnya Bos\) yang ingin menjual “decision power” dalam perusahaannya kepada beberapa investor. Kekuatan keputusan ini \(decision power\) bisa mengambil bentuk dari colored coin yang dapat kita gunakan sebagai latihan untuk membuat “Power Coin” \(sebut saja istilah latihan kali ini dengan nama Power Coin\).

Mari kita memulai:

![](../assets/PowerCoin.png)

Katakan saja bahwa ada tiga orang yang tertarik, Satoshi, Alice and Bob. \(Ya, nama ketiganya sering dijadikan contoh untuk banyak hal\)  
Jadi kemudian Bos memutuskan untuk menjual setiap Power Coin dengan nilai seharga 0.1 BTC pada masing-masingnya.

Mari kita mulai mendanai sejumlah uang untuk address`powerCoin`, `satoshi`, `alice` dan `bob`.

```cs
var powerCoin = new Key();
var alice = new Key();
var bob = new Key();
var satoshi = new Key();
var init = new Transaction()
{
    Outputs = 
    {
        new TxOut(Money.Coins(1.0m), powerCoin),
        new TxOut(Money.Coins(1.0m), alice),
        new TxOut(Money.Coins(1.0m), bob),
        new TxOut(Money.Coins(1.0m), satoshi),
    }
};

var repo = new NoSqlColoredTransactionRepository();
repo.Transactions.Put(init);
```

Kita bayangkan jika semisal Alice membeli 2 Power coin, jadi begini cara membuat transaksinya.

![](../assets/Power2Alice.png)

```cs
var issuance = GetCoins(init,powerCoin)
                .Select(c=> new IssuanceCoin(c))
                .ToArray();
var builder = new TransactionBuilder();
var toAlice =
    builder
    .AddCoins(issuance)
    .AddKeys(powerCoin)
    .IssueAsset(alice, new AssetMoney(powerCoin, 2))
    .SetChange(powerCoin)
    .Then()
    .AddCoins(GetCoins(init, alice))
    .AddKeys(alice)
    .Send(alice, Money.Coins(0.2m))
    .SetChange(alice)
    .BuildTransaction(true);
repo.Transactions.Put(toAlice);
```

Singkat kata, powerCoin menerbitkan 2 Power Coins kepada Alice dan mengirimkan kembaliannya kepada dirinya sendiri. Demikian juga, saat Alice mengirim sejumlah 0.2 BTC kepada powerCoin dan mengirimkan jumlah kembalian untuk dirinya sendiri.

Sedangkan **GetCoins** adalah

```cs
private IEnumerable<Coin> GetCoins(Transaction tx, Key owner)
{
    return tx.Outputs.AsCoins().Where(c => c.ScriptPubKey == owner.ScriptPubKey);
}
```

Untuk beberapa alasan, Alice, mungkin ingin menjual beberapa bagian dari voting power miliknya kepada Satoshi.

![](../assets/PowerCoin2.png)

Anda mungkin bisa mencatat bahwa saya melakukan _double spending_ koin dari Alice pada **init** transaksi.  
\_\*\*\_Hal tersebut tidak akan diterima di Blockchain. Namun, kami belum melihat bagaimana untuk mengambil koin yang belum terpakai itu dari Blockchain dengan mudah, jadi bayangkan saja demi latihan kita, pada koin yang bukan merupakan double spent.

Sekarang Alice dan Satoshi telah memiliki beberapa hak suara, mari kita lihat bagaimana Boss dapat menjalankan proses pemungutan suara tersebut.

### Menjalankan voting {#running-a-vote}

By consulting the Blockchain, Boss can at any time know **ScriptPubKeys** which owns Power Coins.  
So he will send Voting Coins to these owner, proportionally to their voting power, in our case, 1 voting coin to Alice and 1 voting coin to Satoshi.

![](../assets/PowerCoin3.png)

First, I need to create some funds for **votingCoin**.

```cs
var votingCoin = new Key();
var init2 = new Transaction()
{
    Outputs = 
    {
        new TxOut(Money.Coins(1.0m), votingCoin),
    }
};
repo.Transactions.Put(init2);
```

Then, issue the voting coins.

```cs
issuance = GetCoins(init2, votingCoin).Select(c => new IssuanceCoin(c)).ToArray();
builder = new TransactionBuilder();
var toVoters =
    builder
    .AddCoins(issuance)
    .AddKeys(votingCoin)
    .IssueAsset(alice, new AssetMoney(votingCoin, 1))
    .IssueAsset(satoshi, new AssetMoney(votingCoin, 1))
    .SetChange(votingCoin)
    .BuildTransaction(true);
repo.Transactions.Put(toVoters);
```

### Vote delegation {#vote-delegation}

The problem is that the vote concern some financial aspect of the business, and Alice is mostly concerned by the marketing aspect.

Her decision is to handout her voting coin to someone she trusts having a better judgment on financial matter. She chooses to delegate her vote to Bob.

![](../assets/PowerCoin4.png)

```cs
var aliceVotingCoin = ColoredCoin.Find(toVoters,repo)
                        .Where(c=>c.ScriptPubKey == alice.ScriptPubKey)
                        .ToArray();
builder = new TransactionBuilder();
var toBob =
    builder
    .AddCoins(aliceVotingCoin)
    .AddKeys(alice)
    .SendAsset(bob, new AssetMoney(votingCoin, 1))
    .BuildTransaction(true);
repo.Transactions.Put(toBob);
```

You can notice that there is no **SetChange** the reason is that the input colored coin is spent entirely, so nothing is left to be returned.

### Voting {#voting}

Imagine that Satoshi is too busy and decide not to vote. Now Bob must express his decision.  
The vote concerns whether the company should ask for a loan to the bank for investing into new production machines.

Boss says on the company’s website:

Send your coins to 1HZwkjkeaoZfTSaJxDw6aKkxp45agDiEzN for yes and to 1F3sAm6ZtwLAUnj7d38pGFxtP3RVEvtsbV for no.

Bob decides that the company should take the loan:

![](../assets/PowerCoin5.png)

```cs
var bobVotingCoin = ColoredCoin.Find(toVoters, repo)
    .Where(c => c.ScriptPubKey == bob.ScriptPubKey)
    .ToArray();

builder = new TransactionBuilder();
var vote =
    builder
    .AddCoins(bobVotingCoin)
    .AddKeys(bob)
    .SendAsset(BitcoinAddress.Create("1HZwkjkeaoZfTSaJxDw6aKkxp45agDiEzN"),
                new AssetMoney(votingCoin, 1))
    .BuildTransaction(true);
```

Now Boss can compute the result of the vote and see 1-Yes 0-No, Yes win, so he takes the loan.  
Every participants can also count the result by themselves.

### Alternative: Use of Ricardian Contract {#alternative-use-of-ricardian-contract}

In the previous exercise, we have supposed that Boss announced the modalities of the vote out of the Blockchain, on the company’s website.

This works great, but Bob need to know that the website exists.

Another solution is to publish the modalities of the vote directly on the Blockchain within an **Asset Definition File**, so some software can automatically get it and present it to Bob.

The only piece of code that would have changed is during the issuance of the Voting Coins to voters.

```cs
issuance = GetCoins(init2, votingCoin).Select(c => new IssuanceCoin(c)).ToArray();
issuance[0].DefinitionUrl = new Uri("http://boss.com/vote01.json");
builder = new TransactionBuilder();
var toVoters =
    builder
    .AddCoins(issuance)
    .AddKeys(votingCoin)
    .IssueAsset(alice, new AssetMoney(votingCoin, 1))
    .IssueAsset(satoshi, new AssetMoney(votingCoin, 1))
    .SetChange(votingCoin)
    .BuildTransaction(true);
repo.Transactions.Put(toVoters);
```

In such case, Bob can see that during the issuance of his voting coin, an **Asset Definition File** was published, which is nothing more than a JSON document whose schema is partially [specified in Open Asset](https://github.com/OpenAssets/open-assets-protocol/blob/master/asset-definition-protocol.mediawiki).The schema can be extended to have information about things like:

* Expiration of the vote
* Destination of the votes for each candidates
* Human friendly description of it

However, imagine that a hacker wants to cheat the vote. He can always modify the json document \(either man in the middle attack, physical access to boss.com, or access to Bob’s machine\) so Bob is tricked and send his vote to the wrong candidate.

Transforming the **Asset Definition File** into a **Ricardian Contract** by signing it would make any modification immediately detectable by Bob’s software. \(See [Proof Of Authenticity](https://github.com/OpenAssets/open-assets-protocol/blob/master/asset-definition-protocol.mediawiki) in the Asset Definition Protocol\)

