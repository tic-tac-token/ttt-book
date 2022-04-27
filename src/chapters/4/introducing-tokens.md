# 🪙 Tokens

## Introducing tokens
Last week we added access control to our game. We now have a basic two-player game that's restricted to specific players authenticated by address. This week, we'll turn this one shot game into a repeated one with support for multiple simultaneous games, and add a leaderboard to track wins over time. We'll also explore our first Ethereum native contract abstraction: the ERC20 token standard.

#### Goals this week
- Support multiple players and multiple games.
- Learn about Solidity structs and mappings.
- Award points to winners and track their balances on a global leaderboard.
- Learn about the ERC20 token standard.
- Refactor our custom leaderboard to use an ERC20 token.

#### Suggested homework
- Read the [ERC-20 token standard](https://eips.ethereum.org/EIPS/eip-20).
- Read a few implementations of ERC20: [OpenZeppelin](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol), [ConsenSys](https://github.com/ConsenSys/Tokens/blob/master/contracts/eip20/EIP20.sol), [Solmate](https://github.com/Rari-Capital/solmate/blob/main/src/tokens/ERC20.sol), [ds-token](https://github.com/dapphub/ds-token/blob/master/src/token.sol).
- Watch [Creating your own ERC20 token in more than 2 hours](https://nanexcool.medium.com/creating-your-own-erc20-token-in-more-than-2-hours-f0846bc34c9c).

