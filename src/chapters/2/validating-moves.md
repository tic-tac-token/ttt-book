# Validating moves

Let's add another validation. How about making sure we can't overwrite a space that's already been marked? Here's an additional test:

```solidity
    function test_cannot_overwrite_marked_space() public {
        ttt.markSpace(0, "X");
        
        vm.expectRevert("Already marked");
        ttt.markSpace(0, "O");
    }
```

Note that we need to put `vm.expectRevert` on the line directly before the call in our test that we expect to error.

Here's the result:

```bash
$ forge test
[⠊] Compiling...
[⠆] Compiling 1 files with 0.8.10
[⠰] Solc finished in 177.31ms
Compiler run successful

Running 6 tests for src/test/TicTacToken.t.sol:TicTacTokenTest
[PASS] test_can_mark_space_with_O() (gas: 33319)
[PASS] test_can_mark_space_with_X() (gas: 32325)
[PASS] test_cannot_mark_space_with_Z() (gas: 12799)
[FAIL. Reason: Call did not revert as expected] test_cannot_overwrite_marked_space() (gas: 40135)
[PASS] test_get_board() (gas: 44622)
[PASS] test_has_empty_board() (gas: 47219)
Test result: FAILED. 5 passed; 1 failed; finished in 1.13ms

Failed tests:
[FAIL. Reason: Call did not revert as expected] test_cannot_overwrite_marked_space() (gas: 40135)

Encountered a total of 1 failing tests, 5 tests succeeded
```

Let's add another require statement. We can even reuse our string comparison helper function:

```solidity
// SPDX-License-Identifier: Apache-2.0
pragma solidity 0.8.10;

contract TicTacToken {
    string[9] public board;

    function getBoard() public view returns (string[9] memory) {
        return board;
    }

    function markSpace(uint256 space, string calldata symbol) public {
        require(_validSymbol(symbol), "Invalid symbol");
        require(_compareStrings(board[space], ""), "Already marked");
        board[space] = symbol;
    }

    function _validSymbol(string calldata symbol) internal pure returns (bool) {
        return _compareStrings(symbol, "X") || _compareStrings(symbol, "O");
    }

    function _compareStrings(string memory a, string memory b) internal pure returns (bool) {
        return keccak256(abi.encodePacked(a)) == keccak256(abi.encodePacked(b));
    }
}
```

Tests should pass:

```bash
$ forge test
[⠊] Compiling...
[⠆] Compiling 2 files with 0.8.10
[⠰] Solc finished in 183.60ms
Compiler run successful

Running 6 tests for src/test/TicTacToken.t.sol:TicTacTokenTest
[PASS] test_can_mark_space_with_O() (gas: 34537)
[PASS] test_can_mark_space_with_X() (gas: 33543)
[PASS] test_cannot_mark_space_with_Z() (gas: 12800)
[PASS] test_cannot_overwrite_marked_space() (gas: 40038)
[PASS] test_get_board() (gas: 44622)
[PASS] test_has_empty_board() (gas: 47219)
Test result: ok. 6 passed; 0 failed; finished in 781.38µs
```

And just like last time, let's extract and name an internal helper:

```solidity
// SPDX-License-Identifier: Apache-2.0
pragma solidity 0.8.10;

contract TicTacToken {
    string[9] public board;

    function getBoard() public view returns (string[9] memory) {
        return board;
    }

    function markSpace(uint256 space, string calldata symbol) public {
        require(_validSymbol(symbol), "Invalid symbol");
        require(_emptySpace(space), "Already marked");
        board[space] = symbol;
    }

    function _emptySpace(uint256 i) internal view returns (bool) {
        return _compareStrings(board[i], "");
    }

    function _validSymbol(string calldata symbol) internal pure returns (bool) {
        return _compareStrings(symbol, "X") || _compareStrings(symbol, "O");
    }

    function _compareStrings(string memory a, string memory b) internal pure returns (bool) {
        return keccak256(abi.encodePacked(a)) == keccak256(abi.encodePacked(b));
    }
}
```

Note that this helper is a `view`, rather than a `pure` function, because it reads from the `board` state variable. We've got a good start on a basic board.
