# Simplifying validations

Now that we know who's calling `markSpace`, we can simplify a few things. For one, we no longer need to pass a symbol to the `markSpace` function. Instead, we can infer the correct symbol from the caller's address. That means some of our tests, like marking a space with an invalid symbol, will now be unnecessary.

This will be another big change all over our tests, but let's go for it:

```solidity
    function test_can_mark_space_with_X() public {
        playerX.markSpace(0);
        assertEq(ttt.board(0), X);
    }

    function test_can_mark_space_with_O() public {
        playerX.markSpace(0);
        playerO.markSpace(1);
        assertEq(ttt.board(1), O);
    }

    function test_cannot_overwrite_marked_space() public {
        playerX.markSpace(0);

        vm.expectRevert("Already marked");
        playerO.markSpace(0);
    }
```

Remember also to remove the `symbol` argument from our mock `User` and the call it delegates to `ttt`:

```solidity
    function markSpace(uint256 space) public {
        vm.prank(_address);
        ttt.markSpace(space);
    }
```

Over in our production code, we can start by getting the caller's symbol based on address and passing this to our existing validations:

```solidity
    function markSpace(uint256 space) public {
        require(_validPlayer(msg.sender), "Unauthorized");
        uint256 symbol = _getSymbol(msg.sender);
        require(_validSymbol(symbol), "Invalid symbol");
        require(_validTurn(symbol), "Not your turn");
        require(_emptySpace(space), "Already marked");
        board[space] = symbol;
        _turns++;
    }

    function _getSymbol(address player) public view returns (uint256) {
        if (player == playerX) return X;
        if (player == playerO) return O;
        return EMPTY;
    }
```

The tests look good:

```bash
$ forge test
Running 25 tests for src/test/TicTacToken.t.sol:TicTacTokenTest
[PASS] test_auth_nonplayer_cannot_mark_space() (gas: 14941)
[PASS] test_auth_playerO_can_mark_space() (gas: 84313)
[PASS] test_auth_playerX_can_mark_space() (gas: 57606)
[PASS] test_can_mark_space_with_O() (gas: 105794)
[PASS] test_can_mark_space_with_X() (gas: 67768)
[PASS] test_cannot_overwrite_marked_space() (gas: 83181)
[PASS] test_checks_for_antidiagonal_win() (gas: 221918)
[PASS] test_checks_for_diagonal_win() (gas: 198005)
[PASS] test_checks_for_horizontal_win() (gas: 196150)
[PASS] test_checks_for_horizontal_win_row2() (gas: 196447)
[PASS] test_checks_for_vertical_win() (gas: 220647)
[PASS] test_contract_owner() (gas: 7661)
[PASS] test_draw_returns_no_winner() (gas: 268766)
[PASS] test_empty_board_returns_no_winner() (gas: 32303)
[PASS] test_game_in_progress_returns_no_winner() (gas: 92478)
[PASS] test_get_board() (gas: 29218)
[PASS] test_has_empty_board() (gas: 32173)
[PASS] test_msg_sender() (gas: 238582)
[PASS] test_non_owner_cannot_reset_board() (gas: 12706)
[PASS] test_owner_can_reset_board() (gas: 32939)
[PASS] test_reset_board() (gas: 157844)
[PASS] test_stores_player_O() (gas: 7749)
[PASS] test_stores_player_X() (gas: 7794)
[PASS] test_symbols_must_alternate() (gas: 70239)
[PASS] test_tracks_current_turn() (gas: 107864)
Test result: ok. 25 passed; 0 failed; finished in 4.00ms
```

However, now that passing an invalid symbol is impossible, we can do a bit more. Let's remove that `require` statement and update the way we check for a `_validTurn`:

```solidity
    function markSpace(uint256 space) public {
        require(_validPlayer(msg.sender), "Unauthorized");
        require(_validTurn(msg.sender), "Not your turn");
        require(_emptySpace(space), "Already marked");
        board[space] = _getSymbol(msg.sender);
        _turns++;
    }

    function _validTurn(address player) internal view returns (bool) {
        return currentTurn() == _getSymbol(player);
    }
```

Tests pass and our refactor is complete!

```bash
$ forge test
Running 25 tests for src/test/TicTacToken.t.sol:TicTacTokenTest
[PASS] test_auth_nonplayer_cannot_mark_space() (gas: 14941)
[PASS] test_auth_playerO_can_mark_space() (gas: 84664)
[PASS] test_auth_playerX_can_mark_space() (gas: 57706)
[PASS] test_can_mark_space_with_O() (gas: 106145)
[PASS] test_can_mark_space_with_X() (gas: 67868)
[PASS] test_cannot_overwrite_marked_space() (gas: 83187)
[PASS] test_checks_for_antidiagonal_win() (gas: 222971)
[PASS] test_checks_for_diagonal_win() (gas: 198807)
[PASS] test_checks_for_horizontal_win() (gas: 196952)
[PASS] test_checks_for_horizontal_win_row2() (gas: 197249)
[PASS] test_checks_for_vertical_win() (gas: 221700)
[PASS] test_contract_owner() (gas: 7661)
[PASS] test_draw_returns_no_winner() (gas: 270170)
[PASS] test_empty_board_returns_no_winner() (gas: 32303)
[PASS] test_game_in_progress_returns_no_winner() (gas: 92578)
[PASS] test_get_board() (gas: 29218)
[PASS] test_has_empty_board() (gas: 32173)
[PASS] test_msg_sender() (gas: 238582)
[PASS] test_non_owner_cannot_reset_board() (gas: 12706)
[PASS] test_owner_can_reset_board() (gas: 32939)
[PASS] test_reset_board() (gas: 158485)
[PASS] test_stores_player_O() (gas: 7749)
[PASS] test_stores_player_X() (gas: 7794)
[PASS] test_symbols_must_alternate() (gas: 70247)
[PASS] test_tracks_current_turn() (gas: 108215)
Test result: ok. 25 passed; 0 failed; finished in 4.10ms
```
