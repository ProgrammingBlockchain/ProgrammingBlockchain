## Is it random enough? {#is-it-random-enough}

When you call **new Key()**, under the hood, you are using a PRNG ([Pseudo-Random-Number-Generator](https://en.wikipedia.org/wiki/Pseudorandom_number_generator)) to generate your private key. On Windows, it uses the **RNGCryptoServiceProvider**, a .NET wrapper around the Windows Crypto API.

On Android, I use the **SecureRandom** class, and in fact, you can use your own implementation with **RandomUtils.Random**.

On iOS, I have not implemented it and you will need to create your own **IRandom** implementation.

For a computer, being random is hard. But the biggest issue is that it is impossible to know if a series of numbers is really random.

If malware modifies your PRNG (and so, can predict the numbers you will generate), you won’t see it until it is too late.

It means that a cross platform and naïve implementation of PRNG (like using the computer’s clock combined with CPU speed) is dangerous. But you won’t see it until it is too late.

For performance reasons, most PRNG works the same way: a random number, called a **Seed**, is chosen, then a predictable formula generates the next number each time you ask for it.

The amount of randomness of the seed is defined by a measure we call **Entropy**, but the amount of **Entropy** also depends on the observer.

Let’s say you generate a seed from your clock time.
And let’s suppose, for simplicity, that your clock has a 1ms resolution. (Actually, it is a bit more - about 15ms.)

If your attacker knows that you generated the key last week, then your seed has
1000 \* 60 \* 60 \* 24 \* 7 = 604800000 possibilities.

For such attacker, the entropy is log<sub>2</sub>(604800000) = 29.17 bits.

And enumerating such a number on my home computer took less than 2 seconds. We call such enumeration “brute forcing”.

However let’s say, you use the clock time + the process id for generating the seed.
Let’s imagine that there are 1024 different process ids.

So now, the attacker needs to enumerate 604800000 \* 1024 possibilities, which take around 2000 seconds.
Now, let’s add the time when I turned on my computer, assuming the attacker knows I turned it on today, it adds 86400000 possibilities.

Now, the attacker needs to enumerate 604800000 \* 1024 \* 86400000 = 5,35088E+19 possibilities.
However, keep in mind that if the attacker has infiltrated my computer, he can get this last piece of info, and bring down the number of possibilities, reducing entropy.

Entropy is measured by **log<sub>2</sub>(possibilities)** and so log<sub>2</sub>(5,35088E+19) = 65 bits.

Is it enough? Probably, assuming your attacker does not know more information about the realm of possibilities used to generate the seed.

But since the hash of a public key is 20 bytes (160 bits), it is smaller than the total universe of the addresses. You might do better.

> **Note:** Adding entropy is linearly harder, cracking entropy is exponentially harder

An interesting way of generating entropy quickly is by incorporating human intervention, such as moving the mouse.

If you don’t completely trust the platform PRNG (which is [not so paranoic](http://android-developers.blogspot.fr/2013/08/some-securerandom-thoughts.html)), you can add entropy to the PRNG output that NBitcoin is using.

```cs
RandomUtils.AddEntropy("hello");
RandomUtils.AddEntropy(new byte[] { 1, 2, 3 });
var nsaProofKey = new Key();
```

What NBitcoin does when you call **AddEntropy(data)** is:
**additionalEntropy = SHA(SHA(data) ^ additionalEntropy)**

Then when you generate a new number:
**result = SHA(PRNG() ^ additionalEntropy)**

## Key Derivation Function {#key-derivation-function}

However, what is most important is not the number of possibilities. It is the time that an attacker would need to successfully break your key. That’s where KDF enters the game.

KDF, or **Key Derivation Function** is a way to have a stronger key, even if your entropy is low.

Imagine that you want to generate a seed, and the attacker knows that there are 10,000,000 possibilities.
Such a seed would be normally cracked pretty easily.

But what if you could make the enumeration slower?
A KDF is a hash function that waste computing resources on purpose.
Here is an example:

```cs
var derived = SCrypt.BitcoinComputeDerivedKey("hello", new byte[] { 1, 2, 3 });
RandomUtils.AddEntropy(derived);
```

Even if your attacker knows that your source of entropy is 5 letters, he will need to run Scrypt to check each possibility, which take 5 seconds on my computer.

The bottom line is: There is nothing paranoid in distrusting a PRNG, and you can mitigate an attack by both adding entropy and also using a KDF.
Keep in mind that an attacker can decrease entropy by gathering information about you or your system.
If you use the timestamp as entropy source, then an attacker can decrease the entropy by knowing you generated the key last week, and that you only use your computer between 9am and 6pm.

In the previous part I talked briefly about a special KDF called **Scrypt**. As I said, the goal of a KDF is to make brute force costly.

So it should be no surprise for you that a standard already exists for encrypting your private key with a password using a KDF. This is [BIP38](http://www.codeproject.com/Articles/775226/NBitcoin-Cryptography-Part).

![](../assets/EncryptedKey.png)

```cs
var privateKey = new Key();
var bitcoinPrivateKey = privateKey.GetWif(Network.Main);
Console.WriteLine(bitcoinPrivateKey); // L1tZPQt7HHj5V49YtYAMSbAmwN9zRjajgXQt9gGtXhNZbcwbZk2r
BitcoinEncryptedSecret encryptedBitcoinPrivateKey = bitcoinPrivateKey.Encrypt("password");
Console.WriteLine(encryptedBitcoinPrivateKey); // 6PYKYQQgx947Be41aHGypBhK6TA5Xhi9TdPBkatV3fHbbKrdDoBoXFCyLK
var decryptedBitcoinPrivateKey = encryptedBitcoinPrivateKey.GetSecret("password");
Console.WriteLine(decryptedBitcoinPrivateKey); // L1tZPQt7HHj5V49YtYAMSbAmwN9zRjajgXQt9gGtXhNZbcwbZk2r

Console.ReadLine();
```

Such encryption is used in two different cases:

*   You do not trust your storage provider (they can get hacked)
*   You are storing the key on the behalf of somebody else (and you do not want to know their key)

If you own your storage, then encrypting at the database level might be enough.

Be careful if your server takes care of decrypting the key, an attacker might attempt to DDOS your server by forcing it to decrypt lots of keys.

Delegate decryption to the ultimate end user when you can.