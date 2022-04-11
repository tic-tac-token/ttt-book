# Refactoring from strings

Strings are easy to read, but they'll be increasingly difficult to work with as our game grows. Before this gets out of control, let's refactor to a different representation and use integer values to represent the state of each square instead.

First the tests. Let's add integer constants at the top of our test contract, so we can refer to each by a named value like `X`, `O`, or `EMPTY` instead of a magic number. 

Remember that the default value of an empty item in an integer array is `0`, so we need to choose nonzero integers to represent X and O in order to distinguish them from an empty square. Let's use 1 for X and 2 for O.

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
}
```

Now we can replace the `""`, `"X"` and `"O"` strings in our tests with these values:

```solidity
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
}
```

We can follow the same pattern in the game code: add some constants at the top and remove our string comparison helper:

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

It's much more concise without string comparisons everywhere. Let's make sure it still works: 

```bash
Running 8 tests for src/test/TicTacToken.t.sol:TicTacTokenTest
[PASS] test_can_mark_space_with_O() (gas: 74806)
[PASS] test_can_mark_space_with_X() (gas: 51096)
[PASS] test_cannot_mark_space_with_Z() (gas: 10709)
[PASS] test_cannot_overwrite_marked_space() (gas: 56634)
[PASS] test_get_board() (gas: 29128)
[PASS] test_has_empty_board() (gas: 31943)
[PASS] test_symbols_must_alternate() (gas: 56515)
[PASS] test_tracks_current_turn() (gas: 76803)
Test result: ok. 8 passed; 0 failed; finished in 1.12ms
```
