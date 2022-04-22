# Requiring a specific caller

We can use `msg.sender` in combination with a `require` statement to check who's calling our contract functions and limit access to specific addresses. Let's start by restricting who's allowed to `resetBoard`. For example, we could update the function to something like this:

```solidity
    function resetBoard() public {
        require(msg.sender == ???, "Unauthorized");
        delete board;
    }
```

What address should we check against in the right hand side of this comparison? Recall that in our test environment, the caller contract address is `0xb4c79dab8f259c7aee6e5b2aa729821864227e84`. We could hardcode it like this:

```solidity
    function resetBoard() public {
        require(
            msg.sender == address(0xb4c79dab8f259c7aee6e5b2aa729821864227e84),
            "Unauthorized"
        );
        delete board;
    }
```

But this won't work if anyone besides our test contract calls `resetBoard`. Instead, we need to set and store the address in a state variable. Something like this:

```solidity
    address public owner;

    constructor(address _owner) {
        owner = _owner;
    }

    function resetBoard() public {
        require(
            msg.sender == owner,
            "Unauthorized"
        );
        delete board;
    }
```

Let's start with a test, then update the implementation. We'll define an `OWNER` address as a constant in our test contract, pass it as an argument to our game during setup, and refer to it in our test.

```solidity
    address internal constant OWNER = address(1);

    function setUp() public {
        ttt = new TicTacToken(OWNER);
    }

    function test_contract_owner() public {
        assertEq(ttt.owner(), OWNER);
    }
```

We'll need to update the constructor first:

```bash
$ forge test
Error:
   0: Compiler run failed
      TypeError: Wrong argument count for function call: 1 arguments given but expected 0.
        --> /Users/ecm/Projects/ttt-book-code/src/test/TicTacToken.t.sol:32:15:
         |
      32 |         ttt = new TicTacToken(OWNER);
         |               ^^^^^^^^^^^^^^^^^^^^^^
```

Let's add a state variable and set it in the constructor:

```solidity
    address public owner;

    constructor(address _owner) {
        owner = _owner;
    }
```

Looks good:

```bash
$ forge test
Running 19 tests for src/test/TicTacToken.t.sol:TicTacTokenTest
[PASS] test_can_mark_space_with_O() (gas: 74789)
[PASS] test_can_mark_space_with_X() (gas: 51124)
[PASS] test_cannot_mark_space_with_Z() (gas: 10687)
[PASS] test_cannot_overwrite_marked_space() (gas: 56705)
[PASS] test_checks_for_antidiagonal_win() (gas: 183739)
[PASS] test_checks_for_diagonal_win() (gas: 161865)
[PASS] test_checks_for_horizontal_win() (gas: 159915)
[PASS] test_checks_for_horizontal_win_row2() (gas: 160241)
[PASS] test_checks_for_vertical_win() (gas: 182570)
[PASS] test_contract_owner() (gas: 7706)
[PASS] test_draw_returns_no_winner() (gas: 227181)
[PASS] test_empty_board_returns_no_winner() (gas: 32236)
[PASS] test_game_in_progress_returns_no_winner() (gas: 75903)
[PASS] test_get_board() (gas: 29218)
[PASS] test_has_empty_board() (gas: 32195)
[PASS] test_msg_sender() (gas: 238455)
[PASS] test_reset_board() (gas: 125168)
[PASS] test_symbols_must_alternate() (gas: 56453)
[PASS] test_tracks_current_turn() (gas: 76749)
Test result: ok. 19 passed; 0 failed; finished in 3.70ms
```

We still haven't added a check against the `msg.sender` in our `resetBoard` function. Let's first add a couple tests. We can use a new cheatcode to manipulate `msg.sender` in our tests. The `vm.prank` cheatcode sets the `msg.sender` address for the following line, similar to how `vm.expectRevert` applies to the immediately following statement.


```solidity
    function test_owner_can_reset_board() public {
        vm.prank(OWNER);
        ttt.resetBoard();
    }

    function test_non_owner_cannot_reset_board() public {
        vm.expectRevert("Unauthorized");
        ttt.resetBoard();
    }
```

In the first test above, we set `msg.sender` to the `OWNER` address before calling `ttt.resetBoard()`. In the second, `msg.sender` will be the address of our test contract, and we expect the call to `resetBoard` to revert.

Our tests fail as expected:

```bash
$ forge test
Running 21 tests for src/test/TicTacToken.t.sol:TicTacTokenTest
[PASS] test_can_mark_space_with_O() (gas: 74767)
[PASS] test_can_mark_space_with_X() (gas: 51124)
[PASS] test_cannot_mark_space_with_Z() (gas: 10665)
[PASS] test_cannot_overwrite_marked_space() (gas: 56705)
[PASS] test_checks_for_antidiagonal_win() (gas: 183739)
[PASS] test_checks_for_diagonal_win() (gas: 161865)
[PASS] test_checks_for_horizontal_win() (gas: 159915)
[PASS] test_checks_for_horizontal_win_row2() (gas: 160241)
[PASS] test_checks_for_vertical_win() (gas: 182548)
[PASS] test_contract_owner() (gas: 7706)
[PASS] test_draw_returns_no_winner() (gas: 227203)
[PASS] test_empty_board_returns_no_winner() (gas: 32347)
[PASS] test_game_in_progress_returns_no_winner() (gas: 75903)
[PASS] test_get_board() (gas: 29218)
[PASS] test_has_empty_board() (gas: 32195)
[PASS] test_msg_sender() (gas: 238455)
[FAIL. Reason: Call did not revert as expected] test_non_owner_cannot_reset_board() (gas: 30817)
[PASS] test_owner_can_reset_board() (gas: 30799)
[PASS] test_reset_board() (gas: 125168)
[PASS] test_symbols_must_alternate() (gas: 56453)
[PASS] test_tracks_current_turn() (gas: 76749)
Test result: FAILED. 20 passed; 1 failed; finished in 3.44ms

Failed tests:
[FAIL. Reason: Call did not revert as expected] test_non_owner_cannot_reset_board() (gas: 30817)
```

Let's update `resetBoard` to add a `require` statement:

```solidity
    function resetBoard() public {
        require(
            msg.sender == owner,
            "Unauthorized"
        );
        delete board;
    }
```

Run the tests once more...

```bash
Running 21 tests for src/test/TicTacToken.t.sol:TicTacTokenTest
[PASS] test_can_mark_space_with_O() (gas: 74767)
[PASS] test_can_mark_space_with_X() (gas: 51124)
[PASS] test_cannot_mark_space_with_Z() (gas: 10676)
[PASS] test_cannot_overwrite_marked_space() (gas: 56705)
[PASS] test_checks_for_antidiagonal_win() (gas: 183739)
[PASS] test_checks_for_diagonal_win() (gas: 161865)
[PASS] test_checks_for_horizontal_win() (gas: 159915)
[PASS] test_checks_for_horizontal_win_row2() (gas: 160241)
[PASS] test_checks_for_vertical_win() (gas: 182548)
[PASS] test_contract_owner() (gas: 7706)
[PASS] test_draw_returns_no_winner() (gas: 227203)
[PASS] test_empty_board_returns_no_winner() (gas: 32347)
[PASS] test_game_in_progress_returns_no_winner() (gas: 75903)
[PASS] test_get_board() (gas: 29218)
[PASS] test_has_empty_board() (gas: 32195)
[PASS] test_msg_sender() (gas: 238455)
[PASS] test_non_owner_cannot_reset_board() (gas: 12706)
[PASS] test_owner_can_reset_board() (gas: 32939)
[FAIL. Reason: Unauthorized] test_reset_board() (gas: 147658)
[PASS] test_symbols_must_alternate() (gas: 56453)
[PASS] test_tracks_current_turn() (gas: 76749)
Test result: FAILED. 20 passed; 1 failed; finished in 3.25ms

Failed tests:
[FAIL. Reason: Unauthorized] test_reset_board() (gas: 147658)

Encountered a total of 1 failing tests, 20 tests succeeded
```

Our latest two tests both passed, but we've caused our previous `resetBoard` test to fail. We'll have to add a `vm.prank` there, too:

```solidity
    function test_reset_board() public {
        ttt.markSpace(3, X);
        ttt.markSpace(0, O);
        ttt.markSpace(4, X);
        ttt.markSpace(1, O);
        ttt.markSpace(5, X);
        vm.prank(OWNER);
        ttt.resetBoard();
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
```

One more test run and we should be all green again:

```bash
$ forge test
Running 21 tests for src/test/TicTacToken.t.sol:TicTacTokenTest
[PASS] test_can_mark_space_with_O() (gas: 74767)
[PASS] test_can_mark_space_with_X() (gas: 51124)
[PASS] test_cannot_mark_space_with_Z() (gas: 10676)
[PASS] test_cannot_overwrite_marked_space() (gas: 56705)
[PASS] test_checks_for_antidiagonal_win() (gas: 183739)
[PASS] test_checks_for_diagonal_win() (gas: 161865)
[PASS] test_checks_for_horizontal_win() (gas: 159915)
[PASS] test_checks_for_horizontal_win_row2() (gas: 160241)
[PASS] test_checks_for_vertical_win() (gas: 182548)
[PASS] test_contract_owner() (gas: 7706)
[PASS] test_draw_returns_no_winner() (gas: 227203)
[PASS] test_empty_board_returns_no_winner() (gas: 32347)
[PASS] test_game_in_progress_returns_no_winner() (gas: 75903)
[PASS] test_get_board() (gas: 29218)
[PASS] test_has_empty_board() (gas: 32195)
[PASS] test_msg_sender() (gas: 238455)
[PASS] test_non_owner_cannot_reset_board() (gas: 12706)
[PASS] test_owner_can_reset_board() (gas: 32939)
[PASS] test_reset_board() (gas: 130859)
[PASS] test_symbols_must_alternate() (gas: 56453)
[PASS] test_tracks_current_turn() (gas: 76749)
Test result: ok. 21 passed; 0 failed; finished in 4.46ms
```

