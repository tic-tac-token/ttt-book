# The ERC20 standard

ERC20 ([Ethereum Request For Comment Number Twenty](https://eips.ethereum.org/EIPS/eip-20)) is a standardized smart contract interface for fungible tokens on Ethereum. What does that mean?

> **Fungible and nonfungible tokens**
>
> A **token** is a digital asset that can represent just about anything: reputation points, units of currency, shares in a liquidity pool, voting power in a governance system, and much more. Tokens are owned by Ethereum accounts (that is, EOAs or smart contracts), and are typically (but not always!) transferrable from one account to another.
>
> A **fungible** asset is interchangeable and divisible. For example, US dollars, gold, and Ether are all fungible assets. One twenty-dollar bill has the same value as any other, and can be subdivided into dollars and cents. One gold bar is the same as another, and can be subdivided into ounces. One ETH can be used to pay for gas on the Ethereum network the same as any other, and can be subdivided into units as small as one quintillionth of an ETH (called a "wei").
>
> Currencies are not the only type of fungible assets: soybeans, Apple stock, and 1 ounce bags of Cool Ranch Doritos are all fungible assets, too!
>
> A **nonfungible** asset has unique qualities that mean one asset is not interchangeable for another. Things like diamonds, apartments, the Mona Lisa, and JPEG images of 2007 Kia Sedonas are all nonfungible. They all may have value, but every asset is different, since it has specific qualities that vary. Nonfungible tokens (NFTs) exist on Ethereum, too, and have their own standard, [ERC721](https://eips.ethereum.org/EIPS/eip-721), which we'll discuss a little later.

**Why create a standard?**

A standard interface for fungible tokens means application and smart contract developers can depend on a common API and create interoperable contracts, products, and services. Wallets, smart contracts, decentralized finance applications, governance systems, and more all depend on this common interface to read token metadata, access account balances, and transfer from one account to another.

Many of the most popular digital assets are ERC20 tokens. You can see many of them [on Etherscan](https://etherscan.io/tokens). A few examples from this list:

- Stablecoins that are designed to remain pegged 1:1 with the dollar like DAI, USDC, and Tether.
- Wrapped Bitcoin, or wBTC, an ERC20 representation of Bitcoin on the Ethereum blockchain.
- MATIC and FTM, the native tokens for Polygon and Fantom, two other EVM-compatible networks.
- stETH and cETH, wrapper tokens that represent staked Ether in Lido and Ether deposits in Compound.
- UNI, MKR, and COMP, governance tokens for the Uniswap, Maker, and Compound protocols.
- SHIB, a meme-based dogcoin.
