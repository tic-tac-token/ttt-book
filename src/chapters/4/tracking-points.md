# Tracking points

Rather than just counting wins, let's create a more complex points based leaderboard. We can issue points based on the number of turns it takes to win: a gin in three moves should be worth more than a win in five.

Let's award 300 points for a 3-move win, 200 for a 4-move win, and 100 for a 5-move win:

```solidity
    function test_three_move_win_X() public {
        // x | x | x
        // o | o | .
        // . | . | .
        playerX.markSpace(0, 0);
        playerO.markSpace(0, 3);
        playerX.markSpace(0, 1);
        playerO.markSpace(0, 4);
        playerX.markSpace(0, 2);
        assertEq(ttt.totalPoints(PLAYER_X), 300);
    }

    function test_three_move_win_O() public {
        // x | x | .
        // o | o | o
        // x | . | .
        playerX.markSpace(0, 0);
        playerO.markSpace(0, 3);
        playerX.markSpace(0, 1);
        playerO.markSpace(0, 4);
        playerX.markSpace(0, 6);
        playerO.markSpace(0, 5);
        assertEq(ttt.totalPoints(PLAYER_O), 300);
    }

    function test_four_move_win_X() public {
        // x | x | x
        // o | o | .
        // x | o | .
        playerX.markSpace(0, 0);
        playerO.markSpace(0, 3);
        playerX.markSpace(0, 1);
        playerO.markSpace(0, 4);
        playerX.markSpace(0, 6);
        playerO.markSpace(0, 7);
        playerX.markSpace(0, 2);
        assertEq(ttt.totalPoints(PLAYER_X), 200);
    }

    function test_four_move_win_O() public {
        // x | x | .
        // o | o | o
        // x | o | x
        playerX.markSpace(0, 0);
        playerO.markSpace(0, 3);
        playerX.markSpace(0, 1);
        playerO.markSpace(0, 4);
        playerX.markSpace(0, 6);
        playerO.markSpace(0, 7);
        playerX.markSpace(0, 8);
        playerO.markSpace(0, 5);
        assertEq(ttt.totalPoints(PLAYER_O), 200);
    }

    function test_five_move_win_X() public {
        // x | x | x
        // o | o | x
        // x | o | o
        playerX.markSpace(0, 0);
        playerO.markSpace(0, 3);
        playerX.markSpace(0, 1);
        playerO.markSpace(0, 4);
        playerX.markSpace(0, 6);
        playerO.markSpace(0, 7);
        playerX.markSpace(0, 5);
        playerO.markSpace(0, 8);
        playerX.markSpace(0, 2);
        assertEq(ttt.totalPoints(PLAYER_X), 100);
    }
```

(With only 9 spaces, player O cannot actually win in five moves since we assume they always take the second turn).

We'll track point balances in a mapping just like wins, but increment its value by different amounts. Here's one way we can use the number of `turns` to determine the number of moves:

```solidity
    mapping(address => uint256) public totalPoints;

    function markSpace(uint256 id, uint256 space) public {
        require(_validPlayer(id), "Unauthorized");
        require(_validTurn(id), "Not your turn");
        require(_emptySpace(id, space), "Already marked");
        games[id].board[space] = _getSymbol(id, msg.sender);
        games[id].turns++;
        if(winner(id) != 0) {
            address winnerAddress = _getAddress(id, winner(id));
            totalWins[winnerAddress] += 1;
            totalPoints[winnerAddress] += _pointsEarned(id);
        }
    }

    function _pointsEarned(uint256 id) internal view returns (uint256) {
        uint256 turns = games[id].turns;
        uint256 moves;
        if (winner(id) == X) {
            moves = (turns + 1) / 2;
        }
        if (winner(id) == O) {
            moves = turns / 2;
        }
        return 600 - (moves * 100);
    }
```
