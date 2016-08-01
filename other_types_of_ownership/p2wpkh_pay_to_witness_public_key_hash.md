## P2WPKH \(Pay to Witness Public Key Hash\) {#p2wpkh-pay-to-witness-public-key-hash}

Di tahun 2015, Pieter Wuille memperkenalkan sebuah fitur bitcoin yang disebut dengan **Segregated Witness** atau yang disingkat dengan, **segwit**. Pada dasarnya, Segregated Witness memindah proof of ownership dari bagian **scriptSig** pada sebuah transaksi, kepada sebuah bagian yang disebut dengan **witness** dari sebuah input.

Ada sejumlah alasan yang diungkap mengapa menggunakan skema baru ini. Berikut adalah penjelasannya, anda bisa melihatnya di sini: [https:\/\/bitcoincore.org\/en\/2016\/01\/26\/segwit-benefits\/](https://bitcoincore.org/en/2016/01/26/segwit-benefits/)

* **Third party Malleability Fix:** Pada awalnya, third party bisa merubah id transaksi pada transaksi anda sebelum di konfirmasi. Hal tersebut tidak perlu lagi dilakukan. 
* **Linear sig hash scaling:** Penandatanganan transaksi membutuhan hashing keseluruhan input transaksi. Ini berpotensi terjadi serangan DDoS vector pada transaksi yang besar.
* **Signing of input values:** Jumlah pengeluarkan pada input juga ditandatangani, artinya penandatangan juga tidak bisa memungkiri sejumlah biaya transaksi yang disertakan juga dalam transaksi. 
* **Capacity increase:** Memungkinkan untuk mempunyai transaksi lebih dari 1MB setiap 10 menit, meningkat sekitar 1.75.
* **Fraud proof:** Akan dikembangkan lebih lanjut, namun wallet SPV akan mampu memvalidasi aturan konsensus ketimbang hanya dengan mengikuti rantai block terpanjang saja. 

Sebelum tandatangan transaksi dimasukkan ke kalkulasi id transaksi, itu tidak diperlukan lagi.

![](../assets/segwit.png)

Tanda tangan digital ini menyimpan informasi yang sama seperti pada pengeluaran di P2PKH, namun terletak di witness menggantikan scriptSig. Sementara`scriptPubKey,`adalah hasil modifikasi dari

```
OP_DUP OP_HASH160 0067c8970e65107ffbb436a49edd8cb8eb6b567f OP_EQUALVERIFY OP_CHECKSIG
```

Menjadi

```
0 0067c8970e65107ffbb436a49edd8cb8eb6b567f
```

Setiap node yang masih belum mengupgrade software mereka, akan nampak seperti menempatkannya pada _stack_. Artinya disini pada berbagai`scriptSig` dapat dibelanjakan. Sehingga meski tanpa sebuah _signature_, node lama akan menganggap transaksi tersebut adalah transaksi yang valid. Sedangkan node baru, pertama akan menginterpretasi versi **witness, **dan kedua di "_push_" sebagai **witness program**.

Namun kedua kedua node tersebut masih membutuhkan signature untuk memverifikasi transaksi.

**Pada NBitcoin, output pengeluaran P2WPKH output tidak berbeda dengan P2PKH secara normal.  
Untuk mendapat**`ScriptPubKey`** dan menggunakannya dari sebuah public key, digunakan**`PubKey.WitHash`** menggantikan **`PubKey.Hash`**.**

```cs
var key = new Key();
Console.WriteLine(key.PubKey.WitHash.ScriptPubKey);
```

Outputnya, akan nampak seperti ini

```
0 0067c8970e65107ffbb436a49edd8cb8eb6b567f
```

Penandatanganan traksaksi pengeluaran ini akan dijelaskan pada bahasan “Penggunaan `TransactionBuilder`”, hampir tidak berbeda sebenarnya, pada banyak hal, dari kode penandatanganan output P2PKH.

`witness`, sama pada`scriptSig` dari P2PKH, dan `scriptSig` kosong:

```json
"in": [
{
  "prev_out": 
    {
      "hash": "725497eaef527567a0a18b310bbdd8300abe86f82153a39d2f87fef713dc8177",
      "n": 0
    },
  "scriptSig": "",
  "witness": "3044022079d443be2bd39327f92adf47a34e4b6ad7c82af182c71fe76ccd39743ced58cf0220149de3e8f11e47a989483f371d3799a710a7e862dd33c9bd842c417002a1c32901 0363f24cd2cb27bb35eb2292789ce4244d55ce580218fd81688197d4ec3b005a67"
}
```

Jadi sekali lagi semantic P2WPKH sama dengan semantic dari P2PKH, kecuali pada signature tidak diletakkan pada lokasi yang sama seperti sebelumnya.

