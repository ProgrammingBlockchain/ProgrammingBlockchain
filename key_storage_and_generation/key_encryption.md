## Key Encryption {#key-encryption}

In the previous part I talked quickly about a special KDF called **Scrypt.** As I said, the goal of a KDF is to make brute force costly.

So it should be no surprise for you that a standard already exist for encrypting your private key with a password using a KDF. This is [BIP38](http://www.codeproject.com/Articles/775226/NBitcoin-Cryptography-Part).

L136rry4qogVuBBdcD2WkfsGP5SrdCxDxpcPMx4YtwYErq9mbG7W

6PYQCwhP9uPh7ESZsV8sUidBsroZdXuaJmzfYTS6MkEMRdT3pRJbfvu7Ts

L136rry4qogVuBBdcD2WkfsGP5SrdCxDxpcPMx4YtwYErq9mbG7W

Such encryption is used in two different cases:

*   You don’t trust your storage provider (they can get hacked)
*   You are storing the key on the behalf of somebody else (and you don’t want to know his key)

If you own your storage, then encrypting at the database level might be enough.

Be careful if your server takes care of decrypting the key, an attacker might attempt to DDOS your server by forcing it to decrypt lots of keys.

Delegate decryption to the ultimate user when you can.