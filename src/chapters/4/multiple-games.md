# Multiple games

In its current state, our game supports two player addresses, set permanently at contract construction time. The contract's `owner` can reset the board at any time, but our two players are locked in forever. And it supports only one game at a time. Let's move instead towards a more flexible design that can support many players and many simultaneous games. To do this, we'll need two new Solidity concepts: [mappings](https://docs.soliditylang.org/en/latest/types.html#mapping-types) and [structs](https://docs.soliditylang.org/en/latest/types.html#structs).
