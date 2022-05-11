# Tracking wins

Let's start by tracking the number of wins for each player over time. We'll add a `totalWins` function that returns the total number of games won by address:

```solidity
    function test_playerX_win_count_starts_at_zero() public {
        assertEq(ttt.totalWins(PLAYER_X), 0);
    }

    function test_playerO_win_count_starts_at_zero() public {
        assertEq(ttt.totalWins(PLAYER_O), 0);
    }

    function test_increments_win_count_on_win() public {
        playerX.markSpace(0, 0);
        playerO.markSpace(0, 3);
        playerX.markSpace(0, 1);
        playerO.markSpace(0, 4);
        playerX.markSpace(0, 2);
        assertEq(ttt.totalWins(PLAYER_X), 1);

        ttt.newGame(PLAYER_X, PLAYER_O);
        playerX.markSpace(1, 1);
        playerO.markSpace(1, 2);
        playerX.markSpace(1, 3);
        playerO.markSpace(1, 4);
        playerX.markSpace(1, 5);
        playerO.markSpace(1, 6);
        assertEq(ttt.totalWins(PLAYER_O), 1);
    }
```

We can store the win count using a mapping with the player's `address` as its key and their `uint256` win count as its value. We'll rely on the default getter to create a public `totalWins()` function:

```solidity
mapping(address => uint256) public totalWins;
```

Our first two tests should pass thanks to the mapping's default balue of zero, but we'll need to increment the win count in order to pass the third:

```bash
$ forge test -m win_count
Running 3 tests for src/test/TicTacToken.t.sol:TicTacTokenTest
[FAIL] test_increments_win_count_on_win() (gas: 199302)
Logs:
  Error: a == b not satisfied [uint]
    Expected: 1
      Actual: 0

[PASS] test_playerO_win_count_starts_at_zero() (gas: 7811)
[PASS] test_playerX_win_count_starts_at_zero() (gas: 7845)
Test result: FAILED. 2 passed; 1 failed; finished in 2.65ms

Failed tests:
[FAIL] test_increments_win_count_on_win() (gas: 199302)
```

Let's check for a winner each time we mark a space, and increment the value in the mapping when we find one:

```solidity
    function markSpace(uint256 id, uint256 space) public {
        require(_validPlayer(id), "Unauthorized");
        require(_validTurn(id), "Not your turn");
        require(_emptySpace(id, space), "Already marked");
        games[id].board[space] = _getSymbol(id, msg.sender);
        games[id].turns++;
        if(winner(id) != 0) {
            address winnerAddress = _getAddress(id, winner(id));
            totalWins[winnerAddress] += 1;
        }
    }

    function _getAddress(uint256 id, uint256 symbol) internal view returns (address) {
        if (symbol == X) return games[id].playerX;
        if (symbol == O) return games[id].playerO;
        return address(0);
    }
```

Looking good!

```bash
$ forge test -m win_count
[⠊] Compiling...
[⠔] Compiling 2 files with 0.8.10
[⠑] Solc finished in 414.60ms
Compiler run successful

Running 3 tests for src/test/TicTacToken.t.sol:TicTacTokenTest
[PASS] test_increments_win_count_on_win() (gas: 274823)
[PASS] test_playerO_win_count_starts_at_zero() (gas: 7811)
[PASS] test_playerX_win_count_starts_at_zero() (gas: 7845)
Test result: ok. 3 passed; 0 failed; finished in 3.44ms
```
