## Ricardian contracts {#ricardian-contracts}

** WARNING: This is not really used anymore, this section is for pure curiosity **

This part is a copy of an article I wrote on [Coinprism’s blog](http://blog.coinprism.com/2014/12/10/colored-coins-and-ricardian-contracts/). At the time of this writing, NBitcoin do not have any code related to Ricardian Contracts.

### What is a Ricardian Contract {#what-is-a-ricardian-contract}

Generally, an asset is any object representing rights which can be redeemed to an issuer on specific conditions.

*   A company’s share gives right to dividends.
*   A bond gives right to the principal at maturity, coupons bears interest for every period.
*   A voting token gives right to vote decisions about an entity. (Company, election.)
*   Some mix are possible : A share can also be a voting token for the company’s president election.

Such rights are typically enumerated inside a Contract, and signed by the issuer (and a trusted party if needed, like a notary).

A Ricardian contract is a Contract which is cryptographically signed by the issuer, and cannot be dissociated from the asset.

So the contract cannot be denied, tampered, and is provably signed by the issuer.  
Such contract can be kept confidential between the issuer and the redeemer, or published.

Open Asset can already support all of that without changing the core protocol, and here is how.

### Ricardian Contract inside Open Asset {#ricardian-contract-inside-open-asset}

[Here](http://iang.org/papers/ricardian_contract.html) is the formal definition of a ricardian contract:

1.  A contract offered by an issuer to holders,
2.  for a valuable right held by holders, and managed by the issuer,
3.  easily readable by people (like a contract on paper),
4.  readable by programs (parsable like a database),
5.  digitally signed,
6.  carries the keys and server information, and
7.  allied with a unique and secure identifier.

An AssetId is specified by OpenAsset in such way :

```AssetId = Hash160(ScriptPubKey)```

Let’s make such **ScriptPubKey** a P2SH as:

```ScriptPubKey = OP_HASH160 Hash(RedeemScript) OP_EQUAL```

Where:

```RedeemScript = HASH160(RicardianContract) OP_DROP IssuerScript```

**IssuerScript** refer to a classical P2PKH for a simple issuer, multi sig if issuance need several consents. (Issuer + notary for example.)

It should be noted that from Bitcoin 0.10, IssuerScript is arbitrary and can be anything.

The **RicardianContract** can be arbitrary, and kept private. Whoever holds the contract can prove that it applies to this Asset thanks to the hash in the ScriptPubKey.

But let’s make such RicardianContract discoverable and verifiable by wallet clients with the Asset Definition Protocol.

Let’s assume we are issuing a Voting token for candidate A, B or C.

Let’s add to the Open Asset Marker, the following asset definition url: ```u=http://issuer.com/contract```

In the http://issuer.com/contract page, let’s create the following [Asset Definition File](https://github.com/OpenAssets/open-assets-protocol/blob/8b945ba68a781358947325ac008cdd740c89adb3/asset-definition-protocol.mediawiki):  

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

```RicardianContract = AssetDefinitionFile```

This terminate our RicardianContract implemented in OA.

### Check list {#check-list}

* **A contract offered by an issuer to holders.**  
The contract is hosted by the issuer, unalterable, and signed every time the Issuer issues a new asset,

* **For a valuable right held by holders, and managed by the issuer.**  
The right in this sample is a voting right for candidate A,B,C to redeem before 10 jan 2015.

* **Easily readable by people (like a contract on paper.)**  
The human readable contract is in the contract_url, but the JSON might be enough.

* **Readable by programs, (parsable like a database.)**  
The details of the vote are inside the **AssetDefinitionFile**, in JSON format, the authenticity of the contract is verified by software with the **IssuerScript**, and the hash in the **ScriptPubKey**.

* **Digitally signed.**  
The **ScriptPubKey** is signed when the issuer issues the asset, thus, also the hash of the contract, and by extension, the contract itself.

* **Carries the keys and server. informationIssuerScript** is included in the contract

* **Allied with a unique and secure identifier.**  
The **AssetId** is defined by **Hash(ScriptPubKey)** that can’t be changed and is unique.

### What is it for? {#what-is-it-for}

Without Ricardian Contract, it is easy for a malicious issuer to modify or repudiate an Asset Definition File.

Ricardian Contract enforces non-repudiation, make a contract unalterable, so it facilitate arbitration matter between redeemers and issuers.

Also, since the Asset Definition File can’t be changed, it becomes possible to save it on redeemer’s own storage, preventing rupture of access to the contract by a malicious issuer.