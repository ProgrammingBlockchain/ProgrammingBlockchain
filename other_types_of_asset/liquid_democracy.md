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

Dengan mengkonsultasikannya langsung kepada Blockchain, Bos dapat melihat dan tahu **ScriptPubKeys** yang memiliki Power Coins.  
Jadi dia akan mengirim _Voting Coins_ kepada pemilik, secara proporsional atas hak suara mereka. Dalam hal ini, 1 suara untuk Alice, dan 1 suara untuk Satoshi.

![](../assets/PowerCoin3.png)

Pertama, saya harus membuat beberapa dana untuk **votingCoin**.

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

Lalu, menerbitkan _voting coins_.

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

### Delegasi voting {#vote-delegation}

Masalahnya, pada beberapa hal pemungutan suara tersebut tidak terlepas dari aspek keuangan dan bisnis, Alice memperhatikan akpek pemasarannya.

Keputusannya adalah untuk menaruh voting coin kepada orang lain yang dipercaya lebih sesuai untuk memberikan penilaian yang lebih baik tentang masalah keuangan. Sehingga Alice kemudian memilih untuk mendelegasikan suaranya kepada Bob.

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

Anda dapat melihat bahwa tidak ada **SetChange** alasannya adalah agar input tersebut dapat dihabiskan seluruhnya, sehingga tidak ada lagi sisa yang akan dikembalikan.

### Voting {#voting}

Pada saat voting berlangsung, bayangkan lagi jika ternyata Satoshi terlalu sibuk sehingga ia memutuskan untuk tidak memberikan voting. Sedangkan Bob harus mengungkapkan keputusannya.   
Pada voting tersebut, memutuskan apakah perusahaan harus meminta pinjaman ke bank untuk berinvestasi pada sebuah mesin produksi baru misalnya.

Bos berkata pada website perusahaannya:

Kirimkan koin anda ke address 1HZwkjkeaoZfTSaJxDw6aKkxp45agDiEzN untuk voting **"yes"** dan address 1F3sAm6ZtwLAUnj7d38pGFxtP3RVEvtsbV untuk voting** "no"**.

Bob memutuskan bahwa perusahaan harus mengambil pinjaman:

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

Jadi sekarang bos dapat menghitung hasil pemungutan suara, dan melihat hasil ada 1-Yes 0-No, maka "Yes" menjadi pemenangnya, lalu ia mengambil pinjaman.   
Setiap partisipan bisa juga melihat dan menghitung hasil pemungutan suara tersebut.

### Alternatif: Menggunakan Kontrak Ricardian {#alternative-use-of-ricardian-contract}

Pada latihan sebelumnya, kita menyangka bahwa Bos akan mengumumkan modalitas suara secara langsung dari Blockchain, kepada website perusahaannya.

Ini karya yang besar, tetapi Bob perlu mengetahui bahwa situs tersebut memang ada.

Solusi lain untuk mempublikasikan modalitas suara secara langsung pada Blockchain dalam **Asset Definition File**, sehingga beberapa software dapat secara otomatis melihat dan menyampaikannya kepada Bob.

Satu-satunya bagian dari kote yang akan berubah saat penerbitan Voting Coins kepada para pemilih adalah:

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

Dalam hal ini, Bob dapat melihat bahwa selama penerbitan koin voting, **Asset Definition File** telah dipublikasikan, yang tidak lebih dari sebuah dokumen JSON dengan skema yang sebagian ditentukan dalam [Open Asset](https://github.com/OpenAssets/open-assets-protocol/blob/master/asset-definition-protocol.mediawiki). Skema ini dapat diperpanjang agar bisa memiliki beberapa hal seperti:

* Masa berakhirnya pemungutan suara
* Tujuan masing-masing kandidat
* Deskripsi yang lebih _Human friendly \(bisa dibaca manusia\)_

Masih belum berakhir, coba bayangkan jika ternyata ada seorang hacker yang ingin mencurangi pemungutan suara tersebut. Dia mungkin dapat memodifikasi file dokumen json tersebut \(misal dengan middle attack, akses fisik langsung di website perusahaan bos, misalnya di boss.com, atau mengakses lewat perangkat yang dimiliki Bob\) jadi Bob akan bisa ditipu dan ia telah memberikan voting pada kandidat yang salah. 

Mengubah **Asset Definition File** menjadi **Ricardian Contract** dengan menandatanganinya, akan membuat modifikasi itu bisa segera terdeteksi oleh perangkat luanak Bob. \(Lihat [Proof Of Authenticity](https://github.com/OpenAssets/open-assets-protocol/blob/master/asset-definition-protocol.mediawiki) dalam Asset Definition Protocol\)

