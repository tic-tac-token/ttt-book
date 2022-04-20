# Deploying the basic board contract

To explore the EVM paradigm a bit further, let's deploy our game contract to the [Rinkeby test network](https://www.rinkeby.io/), where we'll be able to interact with it in an environment similar to the real world.

> **Test Networks**
>
> Rinkeby is one of several Ethereum [test networks](https://ethereum.org/en/developers/docs/networks/), which are sort of like staging environments for the main Ethereum blockchain. Test networks are used both to test smart contract application code and to test changes to the underlying protocol and client software. Different testnets have slightly different properties, but they mostly work like the real world.

To deploy our contract, we'll need a few things: an account to deploy from, some testnet ether to pay for gas, and an RPC endpoint to submit our transaction. Let's walk through setting up all three.
