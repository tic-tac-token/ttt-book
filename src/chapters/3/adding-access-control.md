# Adding access control

It's great that we can interact with our Tic Tac Toe game contract, but hopefully you can already see the shortcomings of our current code: everyone else can interact withour contract, too! Anyone can mark a square on the board or clobber it altogether at any time by calling our contract's public functions. Rather than a public free-for-all, we'll want to restrict who's allowed to call the functions on our contract.

Fortunately, another property of the weird and wonderful Ethereum programming paradigm makes this possible. Since every state changing function call is a cryptographically signed transaction, we can tell _exactly_ who's calling a function! (Or at least, identify their public key/address). Solidity makes this easy by providing a special global variable called `msg.sender`.

> **Wallets, accounts, addresses, and keys**
>
> An _account_ is an entity with an ether balance that can send Ethereum transactions. There are two types of accounts: externally owned accounts and contracts. Every externally owned account (or "EOA") has a corresponding public and private _key_, and is controlled by the owner of its private key. Contract accounts do not have keys and are instead controlled by their deployed code.
>
> Every account has an _address_, a unique identifier made up of 42 hexadecimal characters, like `0xeD43C19583204FB9eFd041a4d9787bbE5c1965C3`. For an externally owned account, this address is derived from its public key. For a contract account, it's derived from the address of the contract creator and their transaction history. An address is not a public key, but it corresponds to one.
>
> A _wallet_ is a software application for managing an Ethereum account. It typically handles generating a keypair and corresponding address and securely storing the private key for one or many accounts.
