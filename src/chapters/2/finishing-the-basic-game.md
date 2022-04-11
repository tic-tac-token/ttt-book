# Finishing the basic game

Here's the final production code for our basic game:

```solidity
// SPDX-License-Identifier: Apache-2.0
pragma solidity 0.8.10;

contract TicTacToken {
    uint256[9] public board;
    
    uint256 internal constant EMPTY = 0;
    uint256 internal constant X = 1;
    uint256 internal constant O = 2;
    uint256 internal _turns;

    function getBoard() public view returns (uint256[9] memory) {
        return board;
    }

    function markSpace(uint256 space, uint256 symbol) public {
        require(_validSymbol(symbol), "Invalid symbol");
        require(_validTurn(symbol), "Not your turn");
        require(_emptySpace(space), "Already marked");
        board[space] = symbol;
        _turns++;
    }

    function currentTurn() public view returns (uint256) {
        return (_turns % 2 == 0) ? X : O;
    }

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

    function _row(uint256 row) internal view returns (uint256) {
        require(row < 3, "Invalid row");
        uint256 idx = 3 * row;
        return board[idx] * board[idx+1] * board[idx+2];
    }

    function _col(uint256 col) internal view returns (uint256) {
        require(col < 3, "Invalid column");
        return board[col] * board[col+3] * board[col+6];
    }

    function _diag() internal view returns (uint256) {
        return board[0] * board[4] * board[8];
    }
    
    function _antiDiag() internal view returns (uint256) {
        return board[2] * board[4] * board[6];
    }


    function _validTurn(uint256 symbol) internal view returns (bool) {
        return symbol == currentTurn();
    }

    function _emptySpace(uint256 i) internal view returns (bool) {
        return board[i] == EMPTY;
    }

    function _validSymbol(uint256 symbol) internal pure returns (bool) {
        return symbol == X || symbol == O;
    }
}
```

And the full code for our unit tests:

```solidity
// SPDX-License-Identifier: Apache-2.0
pragma solidity 0.8.10;

import "ds-test/test.sol";
import "forge-std/Vm.sol";

import "../TicTacToken.sol";

contract TicTacTokenTest is DSTest {
  Vm internal vm = Vm(HEVM_ADDRESS);
  TicTacToken internal ttt;

  uint256 internal constant EMPTY = 0;
  uint256 internal constant X = 1;
  uint256 internal constant O = 2;

  function setUp() public {
    ttt = new TicTacToken();
  }

  function test_has_empty_board() public {
    for (uint256 i = 0; i < 9; i++) {
      assertEq(ttt.board(i), EMPTY);
    }
  }

  function test_get_board() public {
    uint256[9] memory expected = [
      EMPTY,
      EMPTY,
      EMPTY,
      EMPTY,
      EMPTY,
      EMPTY,
      EMPTY,
      EMPTY,
      EMPTY
    ];
    uint256[9] memory actual = ttt.getBoard();

    for (uint256 i = 0; i < 9; i++) {
      assertEq(actual[i], expected[i]);
    }
  }

  function test_can_mark_space_with_X() public {
    ttt.markSpace(0, X);
    assertEq(ttt.board(0), X);
  }

  function test_can_mark_space_with_O() public {
    ttt.markSpace(0, X);
    ttt.markSpace(1, O);
    assertEq(ttt.board(1), O);
  }

  function test_cannot_mark_space_with_Z() public {
    vm.expectRevert("Invalid symbol");
    ttt.markSpace(0, 3);
  }

  function test_cannot_overwrite_marked_space() public {
    ttt.markSpace(0, X);

    vm.expectRevert("Already marked");
    ttt.markSpace(0, O);
  }

  function test_symbols_must_alternate() public {
    ttt.markSpace(0, X);
    vm.expectRevert("Not your turn");
    ttt.markSpace(1, X);
  }

  function test_tracks_current_turn() public {
    assertEq(ttt.currentTurn(), X);
    ttt.markSpace(0, X);
    assertEq(ttt.currentTurn(), O);
    ttt.markSpace(1, O);
    assertEq(ttt.currentTurn(), X);
  }

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
  
  function test_checks_for_vertical_win() public {
    ttt.markSpace(1, X);
    ttt.markSpace(0, O);
    ttt.markSpace(2, X);
    ttt.markSpace(3, O);
    ttt.markSpace(4, X);
    ttt.markSpace(6, O);
    assertEq(ttt.winner(), O);
  }
  
  function test_checks_for_diagonal_win() public {
    ttt.markSpace(0, X);
    ttt.markSpace(1, O);
    ttt.markSpace(4, X);
    ttt.markSpace(5, O);
    ttt.markSpace(8, X);
    assertEq(ttt.winner(), X);
  }
  
  function test_checks_for_antidiagonal_win() public {
    ttt.markSpace(1, X);
    ttt.markSpace(2, O);
    ttt.markSpace(3, X);
    ttt.markSpace(4, O);
    ttt.markSpace(5, X);
    ttt.markSpace(6, O);
    assertEq(ttt.winner(), O);
  }
  
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
}
```

This is a great start on our core game logic, but we're missing a few important features. At the moment, anyone interacting with our contract can mark the board, as long as they're making a valid move! In the next chapter, we'll explore this permissionless paradigm in more detail, and look at a few patterns for restricting access to specific user addresses.  
