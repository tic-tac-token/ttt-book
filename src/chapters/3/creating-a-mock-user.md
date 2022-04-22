# Creating a mock user

Once we updated our game to check for authorized player addresses, we broke most of our tests:

```bash
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

Encountered a total of 14 failing tests, 12 tests succeeded
```

We _could_ use a bunch of `vm.prank` cheatcodes to stub out the address before every call to `ttt.markSpace`, but that's pretty verbose. Instead, let's use another common ds-test pattern: creating a mock user.

At the top of our tests, let's create a new `User` contract that stores an internal address at construction time:

```solidity
contract User {

    address internal _address;

    constructor(address address_) {
        _address = address_;
    }
}
```

Next, let's pass in an instance of our game, and the `Vm`:

```solidity
contract User {

    TicTacToken internal ttt;
    Vm internal vm;
    address internal _address;

    constructor(address address_, TicTacToken _ttt, Vm _vm) {
        _address = address_;
        ttt = _ttt;
        vm = _vm;
    }
}
```

Let's give this contract its own `markSpace` function that takes the same arguments and delegates to `ttt`:

```solidity
contract User {

    TicTacToken internal ttt;
    Vm internal vm;
    address internal _address;

    constructor(address address_, TicTacToken _ttt, Vm _vm) {
        _address = address_;
        ttt = _ttt;
        vm = _vm;
    }

    function markSpace(uint256 space, uint256 symbol) public {
        ttt.markSpace(space, symbol);
    }
}
```

Finally, before we call `ttt.markSpace`, let's use `vm.prank` to set the caller address to the internal state variable we set up at construction time:

```solidity
contract User {

    TicTacToken internal ttt;
    Vm internal vm;
    address internal _address;

    constructor(address address_, TicTacToken _ttt, Vm _vm) {
        _address = address_;
        ttt = _ttt;
        vm = _vm;
    }

    function markSpace(uint256 space, uint256 symbol) public {
        vm.prank(_address);
        ttt.markSpace(space, symbol);
    }
}
```

We've created a helper contract that can act as a mock user in our tests. Let's create variables for `playerX` and `playerO` at the top of our tests and create them in the `setUp` method:

```solidity
    User internal playerX;
    User internal playerO;

    function setUp() public {
        ttt = new TicTacToken(OWNER, PLAYER_X, PLAYER_O);
        playerX = new User(PLAYER_X, ttt, vm);
        playerO = new User(PLAYER_O, ttt, vm);
    }
```

Now, rather than call `vm.prank` over and over, we can use our helper contracts to simulate user calls instead:

```solidity
    function test_can_mark_space_with_X() public {
        playerX.markSpace(0, X);
        assertEq(ttt.board(0), X);
    }

    function test_can_mark_space_with_O() public {
        playerX.markSpace(0, X);
        playerO.markSpace(1, O);
        assertEq(ttt.board(1), O);
    }

    function test_cannot_mark_space_with_Z() public {
        vm.expectRevert("Invalid symbol");
        playerX.markSpace(0, 3);
    }

    function test_cannot_overwrite_marked_space() public {
        playerX.markSpace(0, X);

        vm.expectRevert("Already marked");
        playerO.markSpace(0, O);
    }

    function test_symbols_must_alternate() public {
        playerX.markSpace(0, X);
        vm.expectRevert("Not your turn");
        playerO.markSpace(1, X);
    }

    function test_tracks_current_turn() public {
        assertEq(ttt.currentTurn(), X);
        playerX.markSpace(0, X);
        assertEq(ttt.currentTurn(), O);
        playerO.markSpace(1, O);
        assertEq(ttt.currentTurn(), X);
    }
```

...hopefully you get the idea: throughout our tests, we'll update `ttt` to `playerX` or `playerO` if we need the call to originate from a specific player.

Now that our tests are fixed up, let's take on a refactor and simplify some of our validations.
