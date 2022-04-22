# Requiring authorized players

Let's follow the same pattern as the contract `owner` and store authorized addresses for player X and player O. We can start by passing these as constructor arguments, too.

First, two new tests:


```solidity
    address internal constant PLAYER_X = address(2);
    address internal constant PLAYER_O = address(3);

    function setUp() public {
        ttt = new TicTacToken(OWNER, PLAYER_X, PLAYER_O);
    }

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

Now that we're storing these addresses, let's limit access to `markSpace`. Just like in `restBoard`, let's check that the caller is one of the authorized addresses. Here are a few tests:

```solidity
    function test_auth_nonplayer_cannot_mark_space() public {
        vm.expectRevert("Unauthorized");
        ttt.markSpace(0, X);
    }

    function test_auth_playerX_can_mark_space() public {
        vm.prank(PLAYER_X);
        ttt.markSpace(0, X);
    }

    function test_auth_playerO_can_mark_space() public {
        vm.prank(PLAYER_X);
        ttt.markSpace(0, X);

        vm.prank(PLAYER_O);
        ttt.markSpace(1, O);
    }
```

To start, let's just run our new tests rather than the whole suite. Before adding a new `require` in `markSpace`, the first one fails, since anyone can mark the board:

```solidity
$ forge test
Running 3 tests for src/test/TicTacToken.t.sol:TicTacTokenTest
[FAIL. Reason: Call did not revert as expected] test_auth_nonplayer_cannot_mark_space() (gas: 55315)
[PASS] test_auth_playerO_can_mark_space() (gas: 79285)
[PASS] test_auth_playerX_can_mark_space() (gas: 55236)
Test result: FAILED. 2 passed; 1 failed; finished in 2.70ms
```

Let's add a new `require` check and a helper function:

```solidity
    function markSpace(uint256 space, uint256 symbol) public {
        require(_validPlayer(msg.sender), "Unauthorized");
        require(_validSymbol(symbol), "Invalid symbol");
        require(_validTurn(symbol), "Not your turn");
        require(_emptySpace(space), "Already marked");
        board[space] = symbol;
        _turns++;
    }

    function _validPlayer(address player) internal view returns (bool) {
        return player == playerX || player == playerO;
    }
```

With our check in place, the tests all pass:

```solidity
Running 3 tests for src/test/TicTacToken.t.sol:TicTacTokenTest
[PASS] test_auth_nonplayer_cannot_mark_space() (gas: 14982)
[PASS] test_auth_playerO_can_mark_space() (gas: 83832)
[PASS] test_auth_playerX_can_mark_space() (gas: 57445)
Test result: ok. 3 passed; 0 failed; finished in 2.14ms
```

Let's run the whole suite:

```bash
$ forge test
Running 26 tests for src/test/TicTacToken.t.sol:TicTacTokenTest
[PASS] test_auth_nonplayer_cannot_mark_space() (gas: 14982)
[PASS] test_auth_playerO_can_mark_space() (gas: 83832)
[PASS] test_auth_playerX_can_mark_space() (gas: 57445)
[FAIL. Reason: Unauthorized] test_can_mark_space_with_O() (gas: 9965)
[FAIL. Reason: Unauthorized] test_can_mark_space_with_X() (gas: 9901)
[FAIL. Reason: Unauthorized] test_cannot_mark_space_with_Z() (gas: 14920)
[FAIL. Reason: Unauthorized] test_cannot_overwrite_marked_space() (gas: 9900)
[FAIL. Reason: Unauthorized] test_checks_for_antidiagonal_win() (gas: 9947)
[FAIL. Reason: Unauthorized] test_checks_for_diagonal_win() (gas: 9899)
[FAIL. Reason: Unauthorized] test_checks_for_horizontal_win() (gas: 9945)
[FAIL. Reason: Unauthorized] test_checks_for_horizontal_win_row2() (gas: 9921)
[FAIL. Reason: Unauthorized] test_checks_for_vertical_win() (gas: 9923)
[PASS] test_contract_owner() (gas: 7728)
[FAIL. Reason: Unauthorized] test_draw_returns_no_winner() (gas: 9900)
[PASS] test_empty_board_returns_no_winner() (gas: 32281)
[FAIL. Reason: Unauthorized] test_game_in_progress_returns_no_winner() (gas: 9937)
[PASS] test_get_board() (gas: 29218)
[PASS] test_has_empty_board() (gas: 32173)
[PASS] test_msg_sender() (gas: 238516)
[PASS] test_non_owner_cannot_reset_board() (gas: 12684)
[PASS] test_owner_can_reset_board() (gas: 32917)
[FAIL. Reason: Unauthorized] test_reset_board() (gas: 9965)
[PASS] test_stores_player_O() (gas: 7727)
[PASS] test_stores_player_X() (gas: 7772)
[FAIL. Reason: Unauthorized] test_symbols_must_alternate() (gas: 9943)
[FAIL. Reason: Unauthorized] test_tracks_current_turn() (gas: 12881)
Test result: FAILED. 12 passed; 14 failed; finished in 3.70ms

Failed tests:
[FAIL. Reason: Unauthorized] test_can_mark_space_with_O() (gas: 9965)
[FAIL. Reason: Unauthorized] test_can_mark_space_with_X() (gas: 9901)
[FAIL. Reason: Unauthorized] test_cannot_mark_space_with_Z() (gas: 14920)
[FAIL. Reason: Unauthorized] test_cannot_overwrite_marked_space() (gas: 9900)
[FAIL. Reason: Unauthorized] test_checks_for_antidiagonal_win() (gas: 9947)
[FAIL. Reason: Unauthorized] test_checks_for_diagonal_win() (gas: 9899)
[FAIL. Reason: Unauthorized] test_checks_for_horizontal_win() (gas: 9945)
[FAIL. Reason: Unauthorized] test_checks_for_horizontal_win_row2() (gas: 9921)
[FAIL. Reason: Unauthorized] test_checks_for_vertical_win() (gas: 9923)
[FAIL. Reason: Unauthorized] test_draw_returns_no_winner() (gas: 9900)
[FAIL. Reason: Unauthorized] test_game_in_progress_returns_no_winner() (gas: 9937)
[FAIL. Reason: Unauthorized] test_reset_board() (gas: 9965)
[FAIL. Reason: Unauthorized] test_symbols_must_alternate() (gas: 9943)
[FAIL. Reason: Unauthorized] test_tracks_current_turn() (gas: 12881)
```

Well, the good news is that our modifier works! The bad news is that we now need to `prank` the address before almost every line of our tests. It's possible to update our tests this way, but let's look at a cleaner pattern: creating a mock user.
