# Validating turns

We're on a roll now. Let's add one final check in our `markSpace` function: symbols must alternate between X and O.

Here's a test:

```solidity
    function test_symbols_must_alternate() public {
        ttt.markSpace(0, "X");
        vm.expectRevert("Not your turn");
        ttt.markSpace(1, "X");
    }
```

And the expected failure:

```bash
Failed tests:
[FAIL. Reason: Call did not revert as expected] test_symbols_must_alternate() (gas: 61452)
```

There are a few clever and efficient ways to track turns, but for now we'll be lazy: let's add an internal `_turns` counter and increment it on every move. Since "X" always goes first, when this counter is even, we'll know it's X's turn and when it's odd, we'll know it's O's:

```solidity
contract TicTacToken {
    string[9] public board;
    uint256 internal _turns;

    function getBoard() public view returns (string[9] memory) {
        return board;
    }

    function markSpace(uint256 space, string calldata symbol) public {
        require(_validSymbol(symbol), "Invalid symbol");
        require(_validTurn(symbol), "Not your turn");
        require(_emptySpace(space), "Already marked");
        board[space] = symbol;
        _turns++;
    }

    function _validTurn(string calldata symbol) internal view returns (bool) {
        return (_turns % 2 == 0) ? _compareStrings(symbol, "X") : _compareStrings(symbol, "O");
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

Our new test, `test_symbols_must_alternate` passed, but it looks like this additional check caused `test_can_mark_space_with_O` to fail:

```bash
Running 7 tests for src/test/TicTacToken.t.sol:TicTacTokenTest
[FAIL. Reason: Not your turn] test_can_mark_space_with_O() (gas: 11050)
[PASS] test_can_mark_space_with_X() (gas: 57093)
[PASS] test_cannot_mark_space_with_Z() (gas: 12832)
[PASS] test_cannot_overwrite_marked_space() (gas: 64932)
[PASS] test_get_board() (gas: 44622)
[PASS] test_has_empty_board() (gas: 47219)
[PASS] test_symbols_must_alternate() (gas: 62462)
Test result: FAILED. 6 passed; 1 failed; finished in 1.25ms

Failed tests:
[FAIL. Reason: Not your turn] test_can_mark_space_with_O() (gas: 11050)

Encountered a total of 1 failing tests, 6 tests succeeded
```

This makes sense: we can no longer have player O move first in this test, since it's X's turn:

```solidity
    function test_can_mark_space_with_O() public {
        ttt.markSpace(0, "O");
        assertEq(ttt.board(0), "O");
    }
```

We'll update the test to have X play a move first:

```solidity
    function test_can_mark_space_with_O() public {
        ttt.markSpace(0, "X");
        ttt.markSpace(1, "O");
        assertEq(ttt.board(1), "O");
    }
```

And we're back to all green:

```bash
Running 7 tests for src/test/TicTacToken.t.sol:TicTacTokenTest
[PASS] test_can_mark_space_with_O() (gas: 85510)
[PASS] test_can_mark_space_with_X() (gas: 57093)
[PASS] test_cannot_mark_space_with_Z() (gas: 12832)
[PASS] test_cannot_overwrite_marked_space() (gas: 64974)
[PASS] test_get_board() (gas: 44622)
[PASS] test_has_empty_board() (gas: 47219)
[PASS] test_symbols_must_alternate() (gas: 62462)
Test result: ok. 7 passed; 0 failed; finished in 1.12ms
```

While we're here, it seems helpful to expose the current turn as a public function. Let's refactor and use `currentTurn` inside our `_validTurn` check. Since `currentTurn` has `public` visibility, we can both call it externally and access it from internal methods:

```solidity
contract TicTacToken {
    string[9] public board;
    uint256 internal _turns;

    function getBoard() public view returns (string[9] memory) {
        return board;
    }

    function markSpace(uint256 space, string calldata symbol) public {
        require(_validSymbol(symbol), "Invalid symbol");
        require(_validTurn(symbol), "Not your turn");
        require(_emptySpace(space), "Already marked");
        board[space] = symbol;
        _turns++;
    }

    function currentTurn() public view returns (string memory) {
        return (_turns % 2 == 0) ? "X" : "O";
    }

    function _validTurn(string calldata symbol) internal view returns (bool) {
        return _compareStrings(symbol, currentTurn());
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

Finally, let's add a test that exercises `currentTurn`:

```solidity
    function test_tracks_current_turn() public {
        assertEq(ttt.currentTurn(), "X");
        ttt.markSpace(0, "X");
        assertEq(ttt.currentTurn(), "O");
        ttt.markSpace(1, "O");
        assertEq(ttt.currentTurn(), "X");
    }
```

Looks good:

```bash
Running 8 tests for src/test/TicTacToken.t.sol:TicTacTokenTest
[PASS] test_can_mark_space_with_O() (gas: 85532)
[PASS] test_can_mark_space_with_X() (gas: 57093)
[PASS] test_cannot_mark_space_with_Z() (gas: 12854)
[PASS] test_cannot_overwrite_marked_space() (gas: 64885)
[PASS] test_get_board() (gas: 44644)
[PASS] test_has_empty_board() (gas: 47219)
[PASS] test_symbols_must_alternate() (gas: 62484)
[PASS] test_tracks_current_turn() (gas: 90235)
Test result: ok. 8 passed; 0 failed; finished in 1.19ms
```
