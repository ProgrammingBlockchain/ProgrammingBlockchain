## Kontrak Ricardian {#ricardian-contracts}

Pada bagian ini adalah sebuah copy artikel yang saya tulis di blog [Coinprism](http://blog.coinprism.com/2014/12/10/colored-coins-and-ricardian-contracts/). Pada saat saya menuliskan artikel ini, NBitcoin masih belum memiliki kode yang berelasi dengan Kontrak Ricardian _**\(Ricardian Contracts\)**_.

### Apa itu Kontrak Ricardian {#what-is-a-ricardian-contract}

Pada umumnya, aset adalah benda yang merepresentasikan hak yang dapat di redem pada beberapa kondisi tertentu.

* Saham perusahaan memberikan hak untuk deviden.
* Obligasi memberikan hak jika telah jatuh tempo, dikenakan bunga untuk setiap periode. 
* Token voting memberikan hak akses suatu entitas tertentu. \(pada perusahaan, atau pemilihan\)
* Memungkinkan memadukan beberapa hal: Saham juga bisa menjadi token voting pada pemilihan presiden perusahaan. 

Hak tersebut biasanya disebutkan dalam kontrak, dan ditandatangani oleh penerbit \(maupun sebuah pihak lain yang dipercaya jika diperlukan, misalnya saja seperti notaris\).

Kontrak Ricardian adalah kontrak cryptografi yang ditandatangani oleh penerbit, dan tidak dapat dipisahkan dari aset.

Jadi kontrak itu tidak bisa ditolak, dirusak, dan ditandatangni oleh penerbit.   
Kontrak tersebut dapat dirahasiakan antara penerbit dan penebusnya, atau juga diterbitkan.

Open Asset dapat mendukung semuanya tanpa harus mengubah protokol inti. Berikut ini caranya.

### Kontrak Ricardian didalam Open Asset {#ricardian-contract-inside-open-asset}

[Berikut adalah definisi formal kontrak Ricardian: ](http://iang.org/papers/ricardian_contract.html)

1. Sebuah kontrak yang ditawarkan oleh penerbit kepada pemegangnya. 
2. Untuk hak berharga yang dimiliki oleh pemegang, di kelola oleh penerbit. 
3. Mudah dibaca oleh orang \(seperti halnya sebuah kontrak di atas kertas\)
4. Dapat dibaca oleh program \(seperti sebuah database\)
5. Ditandatangani secara digital
6. Terdapat key dan informasi server, dan juga 
7. menyatu dengan identifier yang unik dan aman. 

Sebuah AssetId ditentukan oleh OpenAsset seperti pada cara berikut:

`AssetId = Hash160(ScriptPubKey)`

Mari membuat **ScriptPubKey** seperti sebuah P2SH:

`ScriptPubKey = OP_HASH160 Hash(RedeemScript) OP_EQUAL`

Dimana:

`RedeemScript = HASH160(RicardianContract) OP_DROP IssuerScript`

**IssuerScript** mengacu pada P2PKH klasik untuk penerbit sederhana, multi sig issuance perlu beberapa persetujuan \(Issuer + notary seperti pada contoh\).

Perlu dicatat bahwa dari Bitcoin versi 0.10, IssuerScript adalah arbitrary dan bisa apa saja.

**RicardianContract** bisa arbitrary, dan tetap bisa private. Siapapun memegang kontrak dapat membuktikan bahwa itu bisa berlaku untuk aset berkat hash ScriptPubKey.

Mari kita membuat RicardianContract dan diverivikasi oleh wallet klien dengan Asset Definition Protocol.

Kita asumsikan bahwa kita menerbitkan token voting untuk para calon A, B, atau C.

Kita akan menambahkan pada Open Asset Marker, dengan definisi aset url: `u=http://issuer.com/contract`

Di halaman [http:\/\/issuer.com\/contract](http://issuer.com/contract), mari kita membuat [Asset Definition File](https://github.com/OpenAssets/open-assets-protocol/blob/8b945ba68a781358947325ac008cdd740c89adb3/asset-definition-protocol.mediawiki):

```json
{
    "IssuerScript" : IssuerScript,
    "name" : "MyAsset",
    "contract_url" : "http://issuer.com/readableContract",
    "contract_hash" : "DKDKocezifefiouOIUOIUOIufoiez980980",
    "Type" : "Vote",
    "Candidates" : ["A","B","C"],
    "Validity" : "10 jan 2015"
}
```

Dan sekarang kita dapat mendefinisikan Kontrak Ricardian:

`RicardianContract = AssetDefinitionFile`

Hal ini mengakhiri implementasi RicardianContract dalam OA.

### Check list {#check-list}

* **Sebuah kontrak ditawarkan oleh emiten kepada pemegang. ** 
  Kontrak ini diselenggarakan oleh emiten, tidak dapat dirubah, dan ditandatangani setiap kali penerbit mengeluarkan aset baru. 

* **Hak akses berharga yang dimiliki oleh pemegang, dikelola oleh penerbit. **  
  Hak dalam contoh ini adalah hak voting untuk calon A,B,C untuk dapat menebus sebelum 10 jan 2015.

* **Mudah dibaca oleh orang-orang \(seperti sebuah kontrak diatas kertas\)**  
  Kontrak dapat dibaca manusia seperti pada contract\_url, tapi yang mungkin cukup berupa JSON.

* **Dapat dibaca oleh program, \(dapat diparse seperti sebuah database\)**  
  Detail voting berada di dalam **AssetDefinitionFile**, dalam format JSON, keaslian kontrak diverifikasi oleh perangkat lunak dengan **IssuerScript**, dan hash **ScriptPubKey**.

* **Ditandatangani secara digital.**  
  **ScriptPubKey** ditandatangani ketika penerbit mengeluarkan aset, demikian juga dengan hash kontrak tersebut, dan dengan extensi kontrak itu sendiri.

* **Membawa key dan informationIssuerScript** **Server **yang telah termasuk dalam kontrak. 

* **Menyatu dengan identifier yang unik dan aman. **  
  **AssetId** didefinisikan oleh **Hash\(ScriptPubKey\)** yang tidak dapat dirubah dan unik. 


### Apa fungsinya? {#what-is-it-for}

Tanpa Kontrak Ricardian, sangat mudah bagi penerbit perusak \(malicious issuer\) untuk memodifikasi atau menolak Asset Definition File.

Kontrak Ricardian memberlakukan non-repudiation \(tanpa penolakan\), membuat kontrak tidak bisa dirubah, sehingga memudahkan arbitrase antara penebus \(redeemers\) dan penerbit \(issuers\).

Karena Asset Definition File tidak dapat dirubah, memungkinkan untuk menyimpannya pada ruang penyimpanan milik para penebus sendiri, untuk mencegah ditembusnya akses kontrak oleh _malicious issuer_. 

