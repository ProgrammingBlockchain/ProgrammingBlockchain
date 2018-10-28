## Like the good ol’ days {#like-the-good-ol-days}

First, why generate several keys?
The main reason is privacy. Since you can see the balance of all addresses, it is better to use a new address for each transaction.

You can generate a key, like you did at the beginning:

```cs
var privateKey = new Key()
```

However, you have two problems with that:

*   All backups of your wallet that you, have will become outdated when you generate a new key.
*   You cannot delegate the address creation process to an untrusted peer.

If you are developing a web wallet and generate keys on behalf of your users, and one user gets hacked, they will immediately start suspecting you.