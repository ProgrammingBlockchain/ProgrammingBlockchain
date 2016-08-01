## Colored Coins {#colored-coins}

Sampai pembahasan ini, anda juga telah melihat bagaimana mempertukarkan bitcoin di dalam jaringan. Namun anda juga dapat menggunakan jaringan Bitcoin untuk mentransfer dan mempertukarkan berbagai tipe asset. 

Kita menyebut sejumlah asset tersebut dengan “colored coins”.  
Di dalam Blockchain, sejauh ini tidak ada perbedaan antara sebuah koin dan sebuah Colored Coin.

Colored coin diwakili oleh standar **TxOut**. Dalam sebagian besar waktu, beberapa **TxOut** mempunyai nilai Bitcoin yang disebut dengan “Dust”. \(berkisar sekitar 600 satoshi\)

Nilai riil colored coin berada pada **issuer\(penerbit\)** koin tersebut. 

![](../assets/ColoredCoin.png)

Karena colored coin hanyalah sebuah koin standar dalam arti yang khusus, berikut ini adalah yang anda dapat lihat sebagai sebuah bukti kepemilikan. Dan dalam **TransactionBuilder** akan tetap setia untuk melakukannya. Anda dapat mentransfer sebuah colored coin dengan tepat seperti aturan yang kita lihat sebelumnya. 

Blokchain melihat **Colored Coin** sama seperti sebuah **Koin** pada umumnya. 

Anda dapat mewakili berbagai jenis asset dengan colored coin: saham perusahaan, obligasi, stock, maupun voting.

Tidak peduli jenis aset apa yang akan diwakili, akan selalu berelasi dengan kepercayaan antara penerbit aset \(**issuer**\) dan pemiliknya \(**owner**\). 

Jika anda memiliki sejumlah saham perusahaan, perusahaan tersebut mungkin tidak memutuskan untuk mengirim deviden.

Jika anda memiliki sebuah obligasi, maka bank tidak mungkin menukarnya pada saat telah jatuh tempo. 

Namun pelanggaran atas kontrak mungkin akan langsung dapat terdeteksi dengan bantuan dari **Ricardian Contracts**.  
**Ricardian Contract** adalah sebuah kontrak yang ditandatangani oleh pihak penerbit \(issuer\) dengan hak yang telah dilekatkan pada _asset_. Kontrak tersebut dapat dibaca manusia secara normal dalam bentuk \(pdf\), dan juga berupa \(json\), Jadi perangkatnya tersebut dapat secara otomatis akan bisa mendeteksi jika terjadi pelanggaran.  
Penerbit \(**issuer\)** tidak bisa merubah **ricardian contract** yang telah dilekatkan pada sebuah asset.

Dalam hal ini, Blockchain adalah media transprtasi dari sebuah instrumen keuangan saja.   
Inovasi ini, menjelaskan bahwa semua orang dapat membuat dan mampu mentransfer aset sendiri tanpa adanya perantara. Sedangkan pada jenis media transfer aset tradisional seperti _**\(clearing houses\)**_, aturannya cukup berat, dan sengaja dirahasiakan, ditutup untuk masyarakat umum. 

**Open Asset** adalah nama sebuah protokol yang diciptakan oleh Flavien Charlon, menjelaskan bagaimana mentransfer dan memasukkannya ke dalam Blockchain.  
Meski ada beberapa protokol lain, namun Open Asset paling mudah dan fleksibel digunakan, dan juga telah didukung juga di dalam **NBitcoin**.

Pada sisa pembahasan buku ini, saya tidak akan mendetailkan protokol Open Asset, pada halaman GitHub spesifikasinya telah cocok untuk melengkapi kebutuhan tersebut. 

