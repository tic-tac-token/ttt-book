# Checking for wins

To wrap up our basic board, let's add the ability to check for a winner. Let's start with a test for a horizontal win:

```solidity
  function test_checks_for_horizontal_win() public {
    ttt.markSpace(0, X);
    ttt.markSpace(3, O);
    ttt.markSpace(1, X);
    ttt.markSpace(4, O);
    ttt.markSpace(2, X);
    assertEq(ttt.winner(), X);
  }

  function test_checks_for_horizontal_win_row2() public {
    ttt.markSpace(3, X);
    ttt.markSpace(0, O);
    ttt.markSpace(4, X);
    ttt.markSpace(1, O);
    ttt.markSpace(5, X);
    assertEq(ttt.winner(), X);
  }
```

The tests guide us to our next step: we'll need to add a public `winner` function.

```bash
$ forge test
[⠊] Compiling...
[⠒] Compiling 1 files with 0.8.10
[⠢] Solc finished in 10.24ms
Error:
   0: Compiler run failed
      TypeError: Member "winner" not found or not visible after argument-dependent lookup in contract TicTacToken.
        --> /Users/ecm/Projects/ttt-book-code/src/test/TicTacToken.t.sol:89:14:
         |
      89 |     assertEq(ttt.winner(), X);
         |              ^^^^^^^^^^
```

 There are many ways to check for a winner. As we did with `currentTurn`, let's not worry just yet about the most efficient one. (We'll get to that later, when we learn about gas optimization). For now, let's add a helper to get each row, a helper to check if a row has a winner, and a loop to iterate over each row.

We can use some arithmetic to check if a row contains a win by multiplying its three values. If a row is all `X`, its product will be `1 * 1 * 1 = 1`. If it's all `O`, it will be `2 * 2 * 2 = 8`.

Here's a helper to get the product for a row:

 ```solidity
    function _row(uint256 row) internal view returns (uint256) {
        require(row < 3, "Invalid row");
        uint256 idx = 3 * row;
        return board[idx] * board[idx+1] * board[idx+2];
    }
 ```

 And one to check if the product is a win:

```solidity
    function _checkWin(uint256 product) internal pure returns (uint256) {
        if (product == 1) {
            return X;
        }
        if (product == 8) {
            return O;
        }
        return 0;
    }
```

To check for a winner, let's iterate over every combination.

```solidity
    function winner() public view returns (uint256) {
        uint256[3] memory wins = [
            _row(0),
            _row(1),
            _row(2)
        ];
        for (uint256 i; i < wins.length; i++) {
            uint256 win = _checkWin(wins[i]);
            if (win == X || win == O) return win;
        }
        return 0;
    }
```

Not super elegant, but it works:

```bash
Running 9 tests for src/test/TicTacToken.t.sol:TicTacTokenTest
[PASS] test_can_mark_space_with_O() (gas: 74806)
[PASS] test_can_mark_space_with_X() (gas: 51096)
[PASS] test_cannot_mark_space_with_Z() (gas: 10709)
[PASS] test_cannot_overwrite_marked_space() (gas: 56634)
[PASS] test_checks_for_horizontal_win() (gas: 148184)
[PASS] test_checks_for_horizontal_win_row2() (gas: 160329)
[PASS] test_get_board() (gas: 29128)
[PASS] test_has_empty_board() (gas: 31965)
[PASS] test_symbols_must_alternate() (gas: 56537)
[PASS] test_tracks_current_turn() (gas: 76825)
Test result: ok. 9 passed; 0 failed; finished in 1.24ms
```

While we're at it, let's add columns:

```solidity
  function test_checks_for_vertical_win() public {
    ttt.markSpace(1, X);
    ttt.markSpace(0, O);
    ttt.markSpace(2, X);
    ttt.markSpace(3, O);
    ttt.markSpace(4, X);
    ttt.markSpace(6, O);
    assertEq(ttt.winner(), O);
  }
```

We can add another helper and add the columns to our array of winning combinations:

```solidity
    function winner() public view returns (uint256) {
        uint256[6] memory wins = [
            _row(0),
            _row(1),
            _row(2),
            _col(0),
            _col(1),
            _col(2)
        ];
        for (uint256 i; i < wins.length; i++) {
            uint256 win = _checkWin(wins[i]);
            if (win == X || win == O) return win;
        }
        return 0;
    }

    function _col(uint256 col) internal view returns (uint256) {
        require(col < 3, "Invalid column");
        return board[col] * board[col+3] * board[col+6];
    }
```

Finally, we can add the diagonals:

```solidity
    function winner() public view returns (uint256) {
        uint256[8] memory wins = [
            _row(0),
            _row(1),
            _row(2),
            _col(0),
            _col(1),
            _col(2),
            _diag(),
            _antiDiag()
        ];
        for (uint256 i; i < wins.length; i++) {
            uint256 win = _checkWin(wins[i]);
            if (win == X || win == O) return win;
        }
        return 0;
    }

    function _checkWin(uint256 product) internal pure returns (uint256) {
        if (product == 1) {
            return X;
        }
        if (product == 8) {
            return O;
        }
        return 0;
    }

    function _diag() internal view returns (uint256) {
        return board[0] * board[4] * board[8];
    }

    function _antiDiag() internal view returns (uint256) {
        return board[2] * board[4] * board[6];
    }
```

Finally, let's cover our edge cases and make sure games in progress and draws return no winner:

```solidity
  function test_draw_returns_no_winner() public {
    ttt.markSpace(4, X);
    ttt.markSpace(0, O);
    ttt.markSpace(1, X);
    ttt.markSpace(7, O);
    ttt.markSpace(2, X);
    ttt.markSpace(6, O);
    ttt.markSpace(8, X);
    ttt.markSpace(5, O);
    assertEq(ttt.winner(), 0);
  }

  function test_empty_board_returns_no_winner() public {
    assertEq(ttt.winner(), 0);
  }

  function test_game_in_progress_returns_no_winner() public {
    ttt.markSpace(1, X);
    assertEq(ttt.winner(), 0);
  }
```

Everything looks good!

```bash
Running 15 tests for src/test/TicTacToken.t.sol:TicTacTokenTest
[PASS] test_can_mark_space_with_O() (gas: 74788)
[PASS] test_can_mark_space_with_X() (gas: 51123)
[PASS] test_cannot_mark_space_with_Z() (gas: 10687)
[PASS] test_cannot_overwrite_marked_space() (gas: 56749)
[PASS] test_checks_for_antidiagonal_win() (gas: 183587)
[PASS] test_checks_for_diagonal_win() (gas: 161625)
[PASS] test_checks_for_horizontal_win() (gas: 159719)
[PASS] test_checks_for_horizontal_win_row2() (gas: 160329)
[PASS] test_checks_for_vertical_win() (gas: 182374)
[PASS] test_draw_returns_no_winner() (gas: 226962)
[PASS] test_empty_board_returns_no_winner() (gas: 31997)
[PASS] test_game_in_progress_returns_no_winner() (gas: 75509)
[PASS] test_get_board() (gas: 29151)
[PASS] test_has_empty_board() (gas: 31988)
[PASS] test_symbols_must_alternate() (gas: 56519)
[PASS] test_tracks_current_turn() (gas: 76815)
Test result: ok. 15 passed; 0 failed; finished in 1.59ms
```
