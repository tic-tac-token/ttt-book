# Simultaneous games

We've refactored our representation of game state to use a mapping of structs, and updated our interface to take game ID as a parameter. All our tests pass, but before we move on, why don't we add one to make sure our games are really isolated? We can play out two games with different IDs and ensure they have different winners:


```solidity
    function test_games_are_isolated() public {
        playerX.markSpace(0, 1);
        playerO.markSpace(0, 0);
        playerX.markSpace(0, 2);
        playerO.markSpace(0, 3);
        playerX.markSpace(0, 4);
        playerO.markSpace(0, 6);

        playerX.markSpace(1, 0);
        playerO.markSpace(1, 1);
        playerX.markSpace(1, 4);
        playerO.markSpace(1, 5);
        playerX.markSpace(1, 8);

        assertEq(ttt.winner(0), O);
        assertEq(ttt.winner(1), X);
    }
```

Run the tests to make sure we're OK to move on...

```bash
$ forge test
Running 24 tests for src/test/TicTacToken.t.sol:TicTacTokenTest
[PASS] test_auth_nonplayer_cannot_mark_space() (gas: 15150)
[PASS] test_auth_playerO_can_mark_space() (gas: 86115)
[PASS] test_auth_playerX_can_mark_space() (gas: 58296)
[PASS] test_can_mark_space_with_O() (gas: 124844)
[PASS] test_can_mark_space_with_X() (gas: 87678)
[PASS] test_cannot_overwrite_marked_space() (gas: 84386)
[PASS] test_checks_for_antidiagonal_win() (gas: 229074)
[PASS] test_checks_for_diagonal_win() (gas: 204050)
[PASS] test_checks_for_horizontal_win() (gas: 202172)
[PASS] test_checks_for_horizontal_win_row2() (gas: 202492)
[PASS] test_checks_for_vertical_win() (gas: 227806)
[PASS] test_contract_owner() (gas: 7696)
[PASS] test_draw_returns_no_winner() (gas: 277769)
[PASS] test_empty_board_returns_no_winner() (gas: 33944)
[PASS] test_game_in_progress_returns_no_winner() (gas: 94845)
[FAIL. Reason: Unauthorized] test_games_are_isolated() (gas: 234454)
[PASS] test_get_board() (gas: 29362)
[PASS] test_non_owner_cannot_reset_board() (gas: 12800)
[PASS] test_owner_can_reset_board() (gas: 33071)
[PASS] test_reset_board() (gas: 161596)
[PASS] test_stores_player_O() (gas: 12242)
[PASS] test_stores_player_X() (gas: 12308)
[PASS] test_symbols_must_alternate() (gas: 71226)
[PASS] test_tracks_current_turn() (gas: 110065)
Test result: FAILED. 23 passed; 1 failed; finished in 3.66ms

Failed tests:
[FAIL. Reason: Unauthorized] test_games_are_isolated() (gas: 234454)

Encountered a total of 1 failing tests, 23 tests succeeded
```

Looks like we have one more change to make to our game contract: we're setting up a single game with ID 0 at contract construction time, without any mechanism for creating  new games. Trying to play the game with ID 1 fails, because it's an empty struct with `playerX` and `playerO` set to `address(0)`:

```solidity
    constructor(address _owner, address _playerX, address _playerO) {
        owner = _owner;
        games[0].playerX = _playerX;
        games[0].playerO = _playerO;
    }
```

Let's pend the previous test and add one to create a `newGame` function. This should take an address for `playerX` and `playerO` and set up a new game.

```solidity
    function test_creates_new_game() public {
        ttt.newGame(address(5), address(6));
        (address playerXAddr, address playerOAddr, uint256 turns) = ttt.game(1);
        assertEq(playerXAddr, address(5));
        assertEq(playerOAddr, address(6));
        assertEq(turns, 0);
    }
```

We'll need a few moving parts to make this pass. Let's keep track of the next game ID in an internal state variable, `_nextGameID`, and increment it when we create a new game. When we create a new game, we'll set the `playerX` and `playerO` values on the `Game` struct inside the `games` mapping. Finally, we can call `newGame` in the constructor to handle game 0:

```solidity
    uint256 internal _nextGameId;

    constructor(address _owner, address _playerX, address _playerO) {
        owner = _owner;
        newGame(_playerX, _playerO);
    }

    function newGame(address playerX, address playerO) public {
        games[_nextGameId].playerX = playerX;
        games[_nextGameId].playerO = playerO;
        _nextGameId++;
    }
```

Run the tests...

```bash
$ forge test
Running 24 tests for src/test/TicTacToken.t.sol:TicTacTokenTest
[PASS] test_auth_nonplayer_cannot_mark_space() (gas: 15150)
[PASS] test_auth_playerO_can_mark_space() (gas: 86137)
[PASS] test_auth_playerX_can_mark_space() (gas: 58318)
[PASS] test_can_mark_space_with_O() (gas: 124910)
[PASS] test_can_mark_space_with_X() (gas: 87745)
[PASS] test_cannot_overwrite_marked_space() (gas: 84430)
[PASS] test_checks_for_antidiagonal_win() (gas: 229206)
[PASS] test_checks_for_diagonal_win() (gas: 204226)
[PASS] test_checks_for_horizontal_win() (gas: 202305)
[PASS] test_checks_for_horizontal_win_row2() (gas: 202580)
[PASS] test_checks_for_vertical_win() (gas: 227938)
[PASS] test_contract_owner() (gas: 7763)
[PASS] test_creates_new_game() (gas: 59124)
[PASS] test_draw_returns_no_winner() (gas: 277945)
[PASS] test_empty_board_returns_no_winner() (gas: 33922)
[PASS] test_game_in_progress_returns_no_winner() (gas: 94845)
[PASS] test_get_board() (gas: 29362)
[PASS] test_non_owner_cannot_reset_board() (gas: 12800)
[PASS] test_owner_can_reset_board() (gas: 33093)
[PASS] test_reset_board() (gas: 161720)
[PASS] test_stores_player_O() (gas: 12242)
[PASS] test_stores_player_X() (gas: 12286)
[PASS] test_symbols_must_alternate() (gas: 71270)
[PASS] test_tracks_current_turn() (gas: 110109)
Test result: ok. 24 passed; 0 failed; finished in 3.83ms
```

Looks good, but let's remove game setup from the constructor altogether:

```solidity
    constructor(address _owner) {
        owner = _owner;
    }
```

Instead, we'll call `newGame` to set up a new game in our test context:

```solidity
    function setUp() public {
        ttt = new TicTacToken(OWNER);
        playerX = new User(PLAYER_X, ttt, vm);
        playerO = new User(PLAYER_O, ttt, vm);
        ttt.newGame(PLAYER_X, PLAYER_O);
    }
```

Finally, we should be able to unpend and run our original test, with one addition: calling `newGame` to create game 1 alongside game 0:

```solidity
    function test_games_are_isolated() public {
        playerX.markSpace(0, 1);
        playerO.markSpace(0, 0);
        playerX.markSpace(0, 2);
        playerO.markSpace(0, 3);
        playerX.markSpace(0, 4);
        playerO.markSpace(0, 6);

        ttt.newGame(PLAYER_X, PLAYER_O);
        playerX.markSpace(1, 0);
        playerO.markSpace(1, 1);
        playerX.markSpace(1, 4);
        playerO.markSpace(1, 5);
        playerX.markSpace(1, 8);

        assertEq(ttt.winner(0), O);
        assertEq(ttt.winner(1), X);
    }
```

```bash
$ forge test
Running 25 tests for src/test/TicTacToken.t.sol:TicTacTokenTest
[PASS] test_auth_nonplayer_cannot_mark_space() (gas: 15150)
[PASS] test_auth_playerO_can_mark_space() (gas: 86148)
[PASS] test_auth_playerX_can_mark_space() (gas: 58318)
[PASS] test_can_mark_space_with_O() (gas: 124910)
[PASS] test_can_mark_space_with_X() (gas: 87745)
[PASS] test_cannot_overwrite_marked_space() (gas: 84430)
[PASS] test_checks_for_antidiagonal_win() (gas: 229206)
[PASS] test_checks_for_diagonal_win() (gas: 204160)
[PASS] test_checks_for_horizontal_win() (gas: 202305)
[PASS] test_checks_for_horizontal_win_row2() (gas: 202602)
[PASS] test_checks_for_vertical_win() (gas: 227938)
[PASS] test_contract_owner() (gas: 7718)
[PASS] test_creates_new_game() (gas: 59124)
[PASS] test_draw_returns_no_winner() (gas: 277945)
[PASS] test_empty_board_returns_no_winner() (gas: 33944)
[PASS] test_game_in_progress_returns_no_winner() (gas: 94867)
[PASS] test_games_are_isolated() (gas: 450603)
[PASS] test_get_board() (gas: 29384)
[PASS] test_non_owner_cannot_reset_board() (gas: 12800)
[PASS] test_owner_can_reset_board() (gas: 33093)
[PASS] test_reset_board() (gas: 161720)
[PASS] test_stores_player_O() (gas: 12242)
[PASS] test_stores_player_X() (gas: 12308)
[PASS] test_symbols_must_alternate() (gas: 71270)
[PASS] test_tracks_current_turn() (gas: 110109)
Test result: ok. 25 passed; 0 failed; finished in 3.41ms
```

Success! We've pulled off a major redesign to support many simultaneous games with different players.

