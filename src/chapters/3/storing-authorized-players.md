# Storing authorized players

Let's follow the same pattern as the contract `owner` and store authorized addresses for player X and player O. We can start by passing these as constructor arguments, too.

First, two new tests:


```solidity
    address internal constant PLAYER_X = address(2);
    address internal constant PLAYER_O = address(3);

    function test_stores_player_X() public {
        assertEq(ttt.playerX(), PLAYER_X);
    }

    function test_stores_player_O() public {
        assertEq(ttt.playerO(), PLAYER_O);
    }
```

Then we can add new state variables and constructor args in our contract:

```solidity
    address public playerX;
    address public playerO;

    constructor(address _owner, address _playerX, address _playerO) {
        owner = _owner;
        playerX = _playerX;
        playerO = _playerO;
    }
```
