# Supporting multiple games

Our contract currently stores the state of a single, global game using several related state variables:

```solidity
    uint256[9] public board;
    address public playerX;
    address public playerO;
    uint256 internal _turns;
```

We can instead use a struct to group these together in a data structure that represents an individual game:

```solidity
struct Game {
    address playerX;
    address playerO;
    uint256[9] board;
    uint256 turns;
}
```

And a mapping to store many instances of a `Game` by an integer ID:

```solidity
mapping(uint256 => Game) gamesById;
```

Finally, we'll need to parameterize our existing functions to take a game ID as an argument. For example, here's our existing `markSpace` function:

```solidity
    function markSpace(uint256 space) public {
        require(_validPlayer(), "Unauthorized");
        require(_validTurn(), "Not your turn");
        require(_emptySpace(space), "Already marked");
        board[space] = _getSymbol(msg.sender);
        _turns++;
    }
```

We'll now need to pass in a game ID, and probably pass it through to some of our internal helpers:

```solidity
    function markSpace(uint256 gameId, uint256 space) public {
        require(_validPlayer(gameId), "Unauthorized");
        require(_validTurn(gameId), "Not your turn");
        require(_emptySpace(gameId, space), "Already marked");
        gamesById[gameId].board[space] = _getSymbol(gameId, msg.sender);
        gamesById[gameId].turns++;
    }
```

Let's approach this change in three steps. First, we'll refactor to use a struct and mapping internally, without changing our external contract interface. Next, we'll update the interface to expect a game ID as an argument. Finally, we'll add functionality to create new games.

