## Proof of Burn dan Reputasi {#proof-of-burn-and-reputation}

Ada pertanyaan yang cukup sederhana: pada market P2P dimana penegakan hukum menjadi cukup mahal, bagaimana partisipan dapat meminimalkan resiko terjadi scam?

OpenBaazar nampaknya [menjadi yang pertama](https://gist.github.com/dionyziz/e3b296861175e0ebea4b) yang mencoba menerapkan proof of burn sebagai penentu reputasi. 

Ada beberapa tanggapan untuk hal itu \(escrow atau notaris \/ arbiter\), namun satu yang akan kita eksplorasi disini adalah yang disebut dengan _**Proof Of Burn**_.

Coba bayangkan saja jika anda adalah seseorang setengah baya yang tinggal di sebuah desa kecil dengan beberapa pedagang lokal.   
Suatu hari, seorang pedagang yang sering berjualan dari desa kedesa datang menjual barang dengan harga yang luar biasa lebih rendah dibandingkan dengan pedangang lokal. 

Namun, sepanjang perjalanan pedangang tersebut, telah sering dikenal karena telah sering menipu orang dengan kualitas produk yang rendah, sehingga karena telah kehilangan reputasi itu, harganya menjadi lebih kecil dan rendah jika dibandingkan dengan pedagang lokal.   
Sementara pedagang lokal telah menginvestasikan pada sebuah toko yang bagus, untuk membangun iklan dan meningkatkan reputasi mereka. Pelanggan yang tidak senang, mungkin dapat dengan mudah menghancurkan mereka. Namun pedagang yang sering berpergian dari desa ke desa, tidak memiliki sebuah toko lokal, hanya sebuah reputasi sementara saja, dan tidak mempunyai jaminan untuk tidak membuat scam orang lain. 

Pada internet, penciptaan identitas dianggap mudah dan murah, bisa dilakukan semua orang. Semua pedagangnya berpotensi menjadi pedangang yang sering bepergian tersebut. 
Soluusi pada provider market adalah dengan mengumpulkan identitas sebenarnya dari para peserta, sehingga dengan begitu, penegakan hukum menjadi memungkinkan untuk dilakukan. 

Jika anda pernah tertipu di Amazon atau Ebay, bank anda kemungkinan besar akan mengembalikan dana anda, karena mereka mempunyai cara untuk dapat menemukan pencuri itu dengan menghubungi pihak Amazon dan Ebay tersebut. 

Pada market P2P murni yang menggunakan Bitcoin, kita tidak mempunyai itu. Jadi jika anda ternyata di tipu, maka anda sudah dipastikan akan kehilangan uang anda.   
Jadi disini, bagaimana pembeli dapat mempercayai pedagang yang sering berpindah tempat tersebut?   
Tanggapan yang memungkinkan atas pertanyaan itu adalah dengan memeriksa seberapa banyak yang telah ia investasikan untuk membangun reputasinya. 

Jadi sebagai seorang pedagang yang berniat baik, tentu berusaha untuk membangun kepercayaan kepada pelanggan. Dan tentu saja hal tersebut membutuhkan biaya yang besar juga, sehingga para pelanggan pun akan melihat hal itu. Jadi inilah definisi dari “investing into your reputation \(berinvestasi untuk reputasi anda\)”.

Katakanlah jika anda telah membakar \(burning\) sejumlah 50 BTC untuk membangun reputasi anda. Dan kemudian ada seorang pelanggan yang ingin membeli barang senilai 2 BTC dari anda. Pelanggan tersebut tentu mempunyai alasan yang percaya bahwa anda tidak akan menipu pelanggan tersebut, karena anda telah menginvestasikan lebih banyak untuk pembangunan reputasi anda. Dan Jumlah investasi anda lebih besar ketimbang dana dari pelanggan itu.   
Dari sisi ekonomi, bukanlah hal yang menguntungkan bagi anda untuk menipu pelanggan tersebut. 

Detail teknisnya pasti akan bervariasi dan mungkin berubah dari waktu ke waktu, tapi dberikut adalah contoh Proof of Burn.

```cs
var alice = new Key();

//Giving some money to alice
var init = new Transaction()
{
    Outputs = 
    {
        new TxOut(Money.Coins(1.0m), alice),
    }
};

var coin = init.Outputs.AsCoins().First();

//Burning the coin
var burn = new Transaction();
burn.Inputs.Add(new TxIn(coin.Outpoint)
{
    ScriptSig = coin.ScriptPubKey
}); //Spend the previous coin

var message = "Burnt for \"Alice Bakery\"";
var opReturn = TxNullDataTemplate
                .Instance
                .GenerateScriptPubKey(Encoding.UTF8.GetBytes(message));
burn.Outputs.Add(new TxOut(Money.Coins(1.0m), opReturn));
burn.Sign(alice, false);

Console.WriteLine(burn);
```

```json
{
  ….
  "in": [
    {
      "prev_out": {
        "hash": "0767b76406dbaa95cc12d8196196a9e476c81dd328a07b30954d8de256aa1e9f",
        "n": 0
      },
      "scriptSig": "304402202c6897714c69b3f794e730e94dd0110c4b15461e221324b5a78316f97c4dffab0220742c811d62e853dea433e97a4c0ca44e96a0358c9ef950387354fbc24b8964fb01 03fedc2f6458fef30c56cafd71c72a73a9ebfb2125299d8dc6447fdd12ee55a52c"
    }
  ],
  "out": [
    {
      "value": "1.00000000",
      "scriptPubKey": "OP_RETURN 4275726e7420666f722022416c6963652042616b65727922"
    }
  ]
}
```

Setelah transaksi itu disimpan di Blockchain, maka transaksi itu menjadi bukti yang tidak dapat terbantahkan, bahwa Alice telah menginvestasikan sejumlah uang untuk toko rotinya.   
Koin dengan `ScriptPubKey OP_RETURN 4275726e7420666f722022416c6963652042616b65727922` tidak mempunyai cara apapun yang bisa digunakan untuk menggunakan koin itu, jadi koin bisa dianggap telah hilang selamanya. 

