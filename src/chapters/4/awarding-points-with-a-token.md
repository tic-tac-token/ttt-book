# Awarding points with a token

Next, we need to actually use our new token from within our Tic Tac Toe game. Let's update our existing tests to call `token.balanceOf` rather than `ttt.totalPoints`. We'll also need to import our new `Token.sol` contract, set it up in our test setup block, and pass its address through to our `TicTacToken` contract:

```solidity
import "../Token.sol";

contract TicTacTokenTest is DSTest {
    Token internal token;

    function setUp() public {
        token = new Token();
        ttt = new TicTacToken(OWNER, address(token));
        playerX = new User(PLAYER_X, ttt, vm);
        playerO = new User(PLAYER_O, ttt, vm);
        ttt.newGame(PLAYER_X, PLAYER_O);
    }

    function test_increments_win_count_on_win() public {
        playerX.markSpace(0, 0);
        playerO.markSpace(0, 3);
        playerX.markSpace(0, 1);
        playerO.markSpace(0, 4);
        playerX.markSpace(0, 2);
        assertEq(token.balanceOf(PLAYER_X), 1);

        ttt.newGame(PLAYER_X, PLAYER_O);
        playerX.markSpace(1, 1);
        playerO.markSpace(1, 2);
        playerX.markSpace(1, 3);
        playerO.markSpace(1, 4);
        playerX.markSpace(1, 5);
        playerO.markSpace(1, 6);
        assertEq(token.balanceOf(PLAYER_O), 1);
    }

    function test_three_move_win_X() public {
        // x | x | x
        // o | o | .
        // . | . | .
        playerX.markSpace(0, 0);
        playerO.markSpace(0, 3);
        playerX.markSpace(0, 1);
        playerO.markSpace(0, 4);
        playerX.markSpace(0, 2);
        assertEq(token.balanceOf(PLAYER_X), 300);
    }

    function test_three_move_win_O() public {
        // x | x | .
        // o | o | o
        // x | . | .
        playerX.markSpace(0, 0);
        playerO.markSpace(0, 3);
        playerX.markSpace(0, 1);
        playerO.markSpace(0, 4);
        playerX.markSpace(0, 6);
        playerO.markSpace(0, 5);
        assertEq(token.balanceOf(PLAYER_O), 300);
    }

    function test_four_move_win_X() public {
        // x | x | x
        // o | o | .
        // x | o | .
        playerX.markSpace(0, 0);
        playerO.markSpace(0, 3);
        playerX.markSpace(0, 1);
        playerO.markSpace(0, 4);
        playerX.markSpace(0, 6);
        playerO.markSpace(0, 7);
        playerX.markSpace(0, 2);
        assertEq(token.balanceOf(PLAYER_X), 200);
    }

    function test_four_move_win_O() public {
        // x | x | .
        // o | o | o
        // x | o | x
        playerX.markSpace(0, 0);
        playerO.markSpace(0, 3);
        playerX.markSpace(0, 1);
        playerO.markSpace(0, 4);
        playerX.markSpace(0, 6);
        playerO.markSpace(0, 7);
        playerX.markSpace(0, 8);
        playerO.markSpace(0, 5);
        assertEq(token.balanceOf(PLAYER_O), 200);
    }

    function test_five_move_win_X() public {
        // x | x | x
        // o | o | x
        // x | o | o
        playerX.markSpace(0, 0);
        playerO.markSpace(0, 3);
        playerX.markSpace(0, 1);
        playerO.markSpace(0, 4);
        playerX.markSpace(0, 6);
        playerO.markSpace(0, 7);
        playerX.markSpace(0, 5);
        playerO.markSpace(0, 8);
        playerX.markSpace(0, 2);
        assertEq(token.balanceOf(PLAYER_X), 100);
    }
```

Let's first update the `TicTacToken` constructor to take the token address as a second argument:

```solidity
    constructor(address _owner, address _token) {
        owner = _owner;
    }
```

Now our tests will fail as expected:

```bash
Failed tests:
[FAIL] test_five_move_win_X() (gas: 468223)
[FAIL] test_four_move_win_O() (gas: 435191)
[FAIL] test_four_move_win_X() (gas: 398714)
[FAIL] test_three_move_win_O() (gas: 365703)
[FAIL] test_three_move_win_X() (gas: 329183)
```

In order to mint tokens from our game contract, we'll need to make an external call to the `Token` contract. We can do so using an [interface](https://docs.soliditylang.org/en/latest/contracts.html#interfaces).

Since our `Token` contract conforms to the ERC20 interface, we can start with the `IERC20` interface provided by OpenZeppelin. Let's define this at the top of our `TicTacToken.sol` contract:

```solidity
import "openzeppelin-contracts/contracts/token/ERC20/IERC20.sol";

interface IToken is IERC20 {}
```

However, since we've added our own `mint` function on top of the standard interface, we'll need to define it ourselves:

```solidity
import "openzeppelin-contracts/contracts/token/ERC20/IERC20.sol";

interface IToken is IERC20 {
    function mint(address account, uint256 amount) external;
}
```

Now we can create and store an instance of this interface from the token address. All together, it should look something like this:

```solidity
import "openzeppelin-contracts/contracts/token/ERC20/IERC20.sol";
interface IToken is IERC20 {
    function mint(address account, uint256 amount) external;
}

contract TicTacToken {
    IToken internal token;

    constructor(address _owner, address _token) {
        owner = _owner;
        token = IToken(_token);
    }
}
```

Finally, we can call it to award points. Rather than updating the internal `totalPoints` mapping, we'll call `token.mint` with the winner address and number of points earned:

```solidity
    function markSpace(uint256 id, uint256 space) public {
        require(_validPlayer(id), "Unauthorized");
        require(_validTurn(id), "Not your turn");
        require(_emptySpace(id, space), "Already marked");
        games[id].board[space] = _getSymbol(id, msg.sender);
        games[id].turns++;
        if(winner(id) != 0) {
            address winnerAddress = _getAddress(id, winner(id));
            totalWins[winnerAddress] += 1;
            token.mint(winnerAddress, _pointsEarned(id));
        }
    }
```

Run the tests...

```bash
Test result: FAILED. 20 passed; 13 failed; finished in 4.65ms

Failed tests:
[FAIL. Reason: Ownable: caller is not the owner] test_checks_for_antidiagonal_win() (gas: 340300)
[FAIL. Reason: Ownable: caller is not the owner] test_checks_for_diagonal_win() (gas: 303879)
[FAIL. Reason: Ownable: caller is not the owner] test_checks_for_horizontal_win() (gas: 296252)
[FAIL. Reason: Ownable: caller is not the owner] test_checks_for_horizontal_win_row2() (gas: 297474)
[FAIL. Reason: Ownable: caller is not the owner] test_checks_for_vertical_win() (gas: 335265)
[FAIL. Reason: Ownable: caller is not the owner] test_five_move_win_X() (gas: 435268)
[FAIL. Reason: Ownable: caller is not the owner] test_four_move_win_O() (gas: 402235)
[FAIL. Reason: Ownable: caller is not the owner] test_four_move_win_X() (gas: 365747)
[FAIL. Reason: Ownable: caller is not the owner] test_games_are_isolated() (gas: 335298)
[FAIL. Reason: Ownable: caller is not the owner] test_increments_win_count_on_win() (gas: 296185)
[FAIL. Reason: Ownable: caller is not the owner] test_reset_board() (gas: 297506)
[FAIL. Reason: Ownable: caller is not the owner] test_three_move_win_O() (gas: 332736)
[FAIL. Reason: Ownable: caller is not the owner] test_three_move_win_X() (gas: 296227)

Encountered a total of 13 failing tests, 26 tests succeeded
```

Eek! Our `onlyOwner` modifier is working as expected, but our tests are failing because our game contract is not the token owner. (By default, the owner of an `Ownable` contract is its deployer account address. In this case, that's our test contract.)

OpenZeppelin `Ownable` provides a `transferOwnership` method that we can call as part of our test setup to reassign ownership to our game contract:

```solidity
    function setUp() public {
        token = new Token();
        ttt = new TicTacToken(OWNER, address(token));
        playerX = new User(PLAYER_X, ttt, vm);
        playerO = new User(PLAYER_O, ttt, vm);
        ttt.newGame(PLAYER_X, PLAYER_O);
        token.transferOwnership(address(ttt));
    }
```

With this change in place, our tests should now pass, and we can remove the now unused `totalPoints` mapping:

```bash
Running 33 tests for src/test/TicTacToken.t.sol:TicTacTokenTest
[PASS] test_auth_nonplayer_cannot_mark_space() (gas: 15194)
[PASS] test_auth_playerO_can_mark_space() (gas: 121029)
[PASS] test_auth_playerX_can_mark_space() (gas: 84760)
[PASS] test_can_mark_space_with_O() (gas: 145822)
[PASS] test_can_mark_space_with_X() (gas: 98209)
[PASS] test_cannot_overwrite_marked_space() (gas: 110965)
[PASS] test_checks_for_antidiagonal_win() (gas: 399654)
[PASS] test_checks_for_diagonal_win() (gas: 362891)
[PASS] test_checks_for_horizontal_win() (gas: 353368)
[PASS] test_checks_for_horizontal_win_row2() (gas: 354906)
[PASS] test_checks_for_vertical_win() (gas: 393355)
[PASS] test_contract_owner() (gas: 7785)
[PASS] test_creates_new_game() (gas: 59164)
[PASS] test_draw_returns_no_winner() (gas: 361524)
[PASS] test_empty_board_returns_no_winner() (gas: 33955)
[PASS] test_five_move_win_X() (gas: 484602)
[PASS] test_four_move_win_O() (gas: 451570)
[PASS] test_four_move_win_X() (gas: 415093)
[PASS] test_game_in_progress_returns_no_winner() (gas: 105331)
[PASS] test_games_are_isolated() (gas: 746305)
[PASS] test_get_board() (gas: 29384)
[PASS] test_increments_win_count_on_win() (gas: 725272)
[PASS] test_non_owner_cannot_reset_board() (gas: 12844)
[PASS] test_owner_can_reset_board() (gas: 33137)
[PASS] test_playerO_win_count_starts_at_zero() (gas: 7800)
[PASS] test_playerX_win_count_starts_at_zero() (gas: 7849)
[PASS] test_reset_board() (gas: 283243)
[PASS] test_stores_player_O() (gas: 12342)
[PASS] test_stores_player_X() (gas: 12341)
[PASS] test_symbols_must_alternate() (gas: 97712)
[PASS] test_three_move_win_O() (gas: 382082)
[PASS] test_three_move_win_X() (gas: 345562)
[PASS] test_tracks_current_turn() (gas: 145010)
Test result: ok. 33 passed; 0 failed; finished in 5.49ms
```

We've created an ERC20 token, assigned ownership to our game, and used it to award points to the winner.
