# 👯‍♂️ Multiplayer

## Supporting multiple players
Last week we built out the core components of our Tic-tac-toe game: marking the board, validating moves, and checking for wins. This week, we'll turn the basic board framework into a multiplayer game, by restricting access to state changing functions to specific, authorized addresses. But first, we'll talk a little bit about what it means to run code on a public blockchain like Ethereum and take a detour to explore the basic board contract we wrote last week on a live test network.

#### Goals this week
- Add authorization to the game, so that moves can only originate from specific authorized addresses.
- Understand the unique properties of running code on a public blockchain.
- Learn about function modifiers, addresses, external vs contract accounts ,and `msg.sender` in Solidity.

#### Suggested homework
- Read [All about modifiers](https://medium.com/coinmonks/solidity-tutorial-all-about-modifiers-a86cf81c14cb).
- Read the Solidity docs on [function modifiers](https://docs.soliditylang.org/en/latest/contracts.html#function-modifiers) and the [address type](https://docs.soliditylang.org/en/latest/types.html?#address).
- Examine the OpenZeppelin [Ownable](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable.sol) and [ReentrancyGuard](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/security/ReentrancyGuard.sol) contracts to see how they make use of modifiers and `msg.sender`.
- Continue working through the [Cryptozombies](https://cryptozombies.io/) exercises at your own pace to learn basic Solidity syntax.
- If you're done with Cryptozombies, start exploring the [Ethernaut](https://ethernaut.openzeppelin.com/) interactive exercises.

