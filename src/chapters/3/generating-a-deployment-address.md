# Generating a deployment address

Foundry's `cast` command line tool includes a handy utility for generating a one time use wallet under the `cast wallet` subcommand. We can use it to generate a new account for deployment.

One important note: once a private key has been exposed in public, like the one we're about to create, you should **never use it again**. If you're following along with these examples, generate your own address and private key.

```bash
$ cast wallet new
Successfully created new keypair.
Address: 0xeD43C19583204FB9eFd041a4d9787bbE5c1965C3
Private Key: 61bc97eb39d98d3103ec4d107906575189f1c7dbebbddcb400a6cccb72e65c53
```

> **Private Key Space**
>
> How can we be sure the wallet we generate is empty? What if we stumble on the same address as someone else? Although this is mathematically possible, it's incredibly unlikely in practice. An Ethereum private key is 256 bits, meaning there are \\(2^{256}\\) possible private keys. As long as your generator is really random, the odds of generating the same key twice are so low that you can safely assume them away. This is not unique to Ethereum: the same property applies for other public/private keypairs, like SSH and PGP.
>
> Visit [keys.lol](https://keys.lol/) for a great visual example of how empty the key space really is.

Hold on to the address and private key we generated because we'll use them in the following steps.
