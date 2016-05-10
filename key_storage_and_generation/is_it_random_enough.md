## Is it random enough? {#is-it-random-enough}

When you call **new Key()**, under the hood, you are using a PRNG (Pseudo-Random-Number-Generator) to generate your private key. On windows, it uses the **RNGCryptoServiceProvider** of Windows.

On Android, I use the **SecureRandom**, and in fact, you can use your own implementation with **RandomUtils.Random**.

On IOS, I have not implemented it and you need to create your **IRandom** implementation.

For a computer, being random is hard. But the biggest issue is that it is impossible to know if a serie of number is really random.

If a malware modifies your PRNG (and so, can predict the numbers you will generate), you won’t see it until it is too late.

It means that a cross platform and naïve implementation of PRNG (like using computer’s clock combined with CPU speed) is dangerous. But you won’t see it until it is too late.

For performance reason, most PRNG works the same way: a random number, called **Seed**, is chosen, then a predictable formulae generates the next numbers each time you ask for it.

The amount of randomness of the seed is defined by a measure we call **Entropy**, but the amount of **Entropy** also depends on the observer.

Let’s say you generate a seed from your clock time.And let’s imagine that your clock has 1ms of resolution. (reality is more ~15ms)

If your attacker knows that you generated the key last week, then your seed has1000 * 60 * 60 * 24 * 7 = 604800000 possibilities

For such attacker, the entropy is LOG(604800000;2) = 29.17 bits.

And enumerating such number on my home computer took less than 2 seconds…We call such enumeration “brute forcing”.

However let’s say, you use the clock time + the process id for generating the seed.Let’s imagine that there are 1024 different process ids.

So now, the attacker needs to enumerate 604800000 * 1024 possibilities, which take around 2000 seconds.Now, let’s add the time when I turned on my computer, assuming the attacker knows I turned it on today, it adds 86400000 possibilities

Now the attacker needs to enumerate 604800000 * 1024 * 86400000 = 5,35088E+19 possibilitiesHowever, keep in mind that if the attacker infiltrate my computer, he can get this last piece of info, and bring down the number of possibilities, reducing entropy.

Entropy is measured by **LOG(possibilities;2)** and so LOG(5,35088E+19; 2) = 65 bits.

Is it enough? Probably. Assuming your attacker does not know more information about the realm of possibilities.

But since the hash of a public key is 20 bytes = 160 bits, it is smaller than the total universe of the addresses. You might do better.

Note: Adding entropy is linearly harder, cracking entropy is exponentially harder

An interesting way of generating entropy quickly is by asking human intervention. (moving the mouse)

If you don’t trust completely the platform PRNG (which is not [so paranoic](http://android-developers.blogspot.fr/2013/08/some-securerandom-thoughts.html)), you can add entropy to the PRNG output that NBitcoin is using.

What NBitcoin does when you call **AddEntropy(data)** is:**additionalEntropy = SHA(SHA(data) ^ additionalEntropy)**

Then when you generate a new number:**result = SHA(PRNG() ^ additionalEntropy)**

### Key Derivation Function {#key-derivation-function}

However, the most important is not the number of possibilities. It is the time that an attacker would need to successfully break your key. That’s where KDF enters the game.

KDF, or **Key Derivation Function** is a way to have a stronger key, even if your entropy is low.

Imagine that you want to generate a seed, and the attacker knows that there are 10.000.000 possibilities.Such a seed would be normally cracked pretty easily.

But what if you could make the enumeration slower?A KDF is a hash function that waste computing resources on purpose.Here is an example:

Even if your attacker knows that your source of entropy is 5 letters, he will need to run Scrypt to check a possibility, which take 5 seconds on my computer.

Bottom line of the story: There is nothing paranoid into distrusting a PRNG, you can mitigate an attack by both adding entropy and also using a KDF.Keep in mind that an attacker can decrease entropy by gathering information about you or your system.If you use the timestamp as entropy source, then he can decrease the entropy by knowingyou generated the key last week, and that you only use your computer between 9am and 6pm.