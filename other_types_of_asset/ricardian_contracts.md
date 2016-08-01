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

**IssuerScript** refer to a classical P2PKH for a simple issuer, multi sig if issuance need several consents. \(Issuer + notary for example.\)

It should be noted that from Bitcoin 0.10, IssuerScript is arbitrary and can be anything.

The **RicardianContract** can be arbitrary, and kept private. Whoever holds the contract can prove that it applies to this Asset thanks to the hash in the ScriptPubKey.

But let’s make such RicardianContract discoverable and verifiable by wallet clients with the Asset Definition Protocol.

Let’s assume we are issuing a Voting token for candidate A, B or C.

Let’s add to the Open Asset Marker, the following asset definition url: `u=http://issuer.com/contract`

In the [http:\/\/issuer.com\/contract](http://issuer.com/contract) page, let’s create the following [Asset Definition File](https://github.com/OpenAssets/open-assets-protocol/blob/8b945ba68a781358947325ac008cdd740c89adb3/asset-definition-protocol.mediawiki):

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

And now we can define the RicardianContract:

`RicardianContract = AssetDefinitionFile`

This terminate our RicardianContract implemented in OA.

### Check list {#check-list}

* **A contract offered by an issuer to holders.**  
  The contract is hosted by the issuer, unalterable, and signed every time the Issuer issues a new asset,

* **For a valuable right held by holders, and managed by the issuer.**  
  The right in this sample is a voting right for candidate A,B,C to redeem before 10 jan 2015.

* **Easily readable by people \(like a contract on paper.\)**  
  The human readable contract is in the contract\_url, but the JSON might be enough.

* **Readable by programs, \(parsable like a database.\)**  
  The details of the vote are inside the **AssetDefinitionFile**, in JSON format, the authenticity of the contract is verified by software with the **IssuerScript**, and the hash in the **ScriptPubKey**.

* **Digitally signed.**  
  The **ScriptPubKey** is signed when the issuer issues the asset, thus, also the hash of the contract, and by extension, the contract itself.

* **Carries the keys and server. informationIssuerScript** is included in the contract

* **Allied with a unique and secure identifier.**  
  The **AssetId** is defined by **Hash\(ScriptPubKey\)** that can’t be changed and is unique.


### What is it for? {#what-is-it-for}

Without Ricardian Contract, it is easy for a malicious issuer to modify or repudiate an Asset Definition File.

Ricardian Contract enforces non-repudiation, make a contract unalterable, so it facilitate arbitration matter between redeemers and issuers.

Also, since the Asset Definition File can’t be changed, it becomes possible to save it on redeemer’s own storage, preventing rupture of access to the contract by a malicious issuer.

