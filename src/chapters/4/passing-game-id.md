# Passing game ID

Next we need to update the functions that make up the external interface of our contract to take a game ID as a parameter.

The good news is that it's easy to find all the places we need to update. Anywhere we're referring to `games[0]` will need to use a function argument instead. The bad news is that there are a lot of them:

```solidity
contract TicTacToken {
    address public owner;

    struct Game {
        address playerX;
        address playerO;
        uint256 turns;
        uint256[9] board;
    }
    mapping(uint256 => Game) public games;

    uint256 internal constant EMPTY = 0;
    uint256 internal constant X = 1;
    uint256 internal constant O = 2;

    constructor(address _owner, address _playerX, address _playerO) {
        owner = _owner;
        games[0].playerX = _playerX;
        games[0].playerO = _playerO;
    }

    function getBoard() public view returns (uint256[9] memory) {
        return games[0].board;
    }

    function resetBoard() public {
        require(
            msg.sender == owner,
            "Unauthorized"
        );
        delete games[0].board;
    }

    function markSpace(uint256 space) public {
        require(_validPlayer(), "Unauthorized");
        require(_validTurn(), "Not your turn");
        require(_emptySpace(space), "Already marked");
        games[0].board[space] = _getSymbol(msg.sender);
        games[0].turns++;
    }

    function currentTurn() public view returns (uint256) {
        return (games[0].turns % 2 == 0) ? X : O;
    }

    function msgSender() public view returns (address) {
        return msg.sender;
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

    function _getSymbol(address player) public view returns (uint256) {
        if (player == games[0].playerX) return X;
        if (player == games[0].playerO) return O;
        return EMPTY;
    }

    function _validTurn() internal view returns (bool) {
        return currentTurn() == _getSymbol(msg.sender);
    }

    function _validPlayer() internal view returns (bool) {
        return msg.sender == games[0].playerX || msg.sender == games[0].playerO;
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
        return games[0].board[idx] * games[0].board[idx + 1] * games[0].board[idx + 2];
    }

    function _col(uint256 col) internal view returns (uint256) {
        require(col < 3, "Invalid column");
        return games[0].board[col] * games[0].board[col + 3] * games[0].board[col + 6];
    }

    function _diag() internal view returns (uint256) {
        return games[0].board[0] * games[0].board[4] * games[0].board[8];
    }

    function _antiDiag() internal view returns (uint256) {
        return games[0].board[2] * games[0].board[4] * games[0].board[6];
    }

    function _validTurn(uint256 symbol) internal view returns (bool) {
        return symbol == currentTurn();
    }

    function _emptySpace(uint256 i) internal view returns (bool) {
        return games[0].board[i] == EMPTY;
    }

    function _validSymbol(uint256 symbol) internal pure returns (bool) {
        return symbol == X || symbol == O;
    }
}
```

We'll also need to update many of our tests, to call these new parameterized functions. Rather than approach this as one sweeping change, it's safer and simpler to work function by function. Let's start with `getBoard`.

First, we'll update the four tests that refer to `getBoard` to pass a game ID argument:

```solidity
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
        uint256[9] memory actual = ttt.getBoard(0);

        for (uint256 i = 0; i < 9; i++) {
            assertEq(actual[i], expected[i]);
        }
    }

    function test_reset_board() public {
        playerX.markSpace(3);
        playerO.markSpace(0);
        playerX.markSpace(4);
        playerO.markSpace(1);
        playerX.markSpace(5);
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
        uint256[9] memory actual = ttt.getBoard(0);

        for (uint256 i = 0; i < 9; i++) {
            assertEq(actual[i], expected[i]);
        }
    }

    function test_can_mark_space_with_X() public {
        playerX.markSpace(0);
        assertEq(ttt.getBoard(0)[0], X);
    }

    function test_can_mark_space_with_O() public {
        playerX.markSpace(0);
        playerO.markSpace(1);
        assertEq(ttt.getBoard(0)[1], O);
    }
```

Tests should now fail with a compiler error:

```bash
$ forge test
[⠊] Compiling...
[⠒] Compiling 1 files with 0.8.10
[⠢] Solc finished in 11.82ms
Error:
   0: Compiler run failed
      TypeError: Wrong argument count for function call: 1 arguments given but expected 0.
        --> /Users/ecm/Projects/ttt-book-code/src/test/TicTacToken.t.sol:71:36:
         |
      71 |         uint256[9] memory actual = ttt.getBoard(0);
         |                                    ^^^^^^^^^^^^^^^



      TypeError: Type is not callable
        --> /Users/ecm/Projects/ttt-book-code/src/test/TicTacToken.t.sol:71:36:
         |
      71 |         uint256[9] memory actual = ttt.getBoard(0);
         |                                    ^^^^^^^^^^^^^^^^^
```

Let's add an `id` argument to `getBoard`:

```solidity
    function getBoard(uint256 id) public view returns (uint256[9] memory) {
        return games[id].board;
    }
```

And finally, we should be back to green:

```bash
$ forge test
Running 24 tests for src/test/TicTacToken.t.sol:TicTacTokenTest
[PASS] test_auth_nonplayer_cannot_mark_space() (gas: 14968)
[PASS] test_auth_playerO_can_mark_space() (gas: 85076)
[PASS] test_auth_playerX_can_mark_space() (gas: 57870)
[PASS] test_can_mark_space_with_O() (gas: 123783)
[PASS] test_can_mark_space_with_X() (gas: 87276)
[PASS] test_cannot_overwrite_marked_space() (gas: 83453)
[PASS] test_checks_for_antidiagonal_win() (gas: 225130)
[PASS] test_checks_for_diagonal_win() (gas: 200762)
[PASS] test_checks_for_horizontal_win() (gas: 198884)
[PASS] test_checks_for_horizontal_win_row2() (gas: 199204)
[PASS] test_checks_for_vertical_win() (gas: 223859)
[PASS] test_contract_owner() (gas: 7696)
[PASS] test_draw_returns_no_winner() (gas: 272697)
[PASS] test_empty_board_returns_no_winner() (gas: 33358)
[PASS] test_game_in_progress_returns_no_winner() (gas: 93797)
[PASS] test_get_board() (gas: 29429)
[PASS] test_msg_sender() (gas: 238687)
[PASS] test_non_owner_cannot_reset_board() (gas: 12706)
[PASS] test_owner_can_reset_board() (gas: 32974)
[PASS] test_reset_board() (gas: 159400)
[PASS] test_stores_player_O() (gas: 12242)
[PASS] test_stores_player_X() (gas: 12308)
[PASS] test_symbols_must_alternate() (gas: 70440)
[PASS] test_tracks_current_turn() (gas: 108670)
Test result: ok. 24 passed; 0 failed; finished in 3.99ms
```

We can follow the same approach for the rest of our public functions: `resetBoard`, `markSpace`, `currentTurn`, and `winner`. The interface to almost every function will have to change, but it won't be too bad if we take small steps and let the compiler guide us towards all the necessary changes.

We won't walk through the full exercise here, but here's a look at our game contract and tests following this big refactor:

The game:

```solidity
// SPDX-License-Identifier: Apache-2.0
pragma solidity 0.8.10;

contract TicTacToken {
    address public owner;

    struct Game {
        address playerX;
        address playerO;
        uint256 turns;
        uint256[9] board;
    }
    mapping(uint256 => Game) public games;

    uint256 internal constant EMPTY = 0;
    uint256 internal constant X = 1;
    uint256 internal constant O = 2;

    constructor(address _owner, address _playerX, address _playerO) {
        owner = _owner;
        games[0].playerX = _playerX;
        games[0].playerO = _playerO;
    }

    function getBoard(uint256 id) public view returns (uint256[9] memory) {
        return games[id].board;
    }

    function resetBoard(uint256 id) public {
        require(
            msg.sender == owner,
            "Unauthorized"
        );
        delete games[id].board;
    }

    function markSpace(uint256 id, uint256 space) public {
        require(_validPlayer(id), "Unauthorized");
        require(_validTurn(id), "Not your turn");
        require(_emptySpace(id, space), "Already marked");
        games[id].board[space] = _getSymbol(id, msg.sender);
        games[id].turns++;
    }

    function currentTurn(uint256 id) public view returns (uint256) {
        return (games[id].turns % 2 == 0) ? X : O;
    }

    function winner(uint256 id) public view returns (uint256) {
        uint256[8] memory wins = [
            _row(id, 0),
            _row(id, 1),
            _row(id, 2),
            _col(id, 0),
            _col(id, 1),
            _col(id, 2),
            _diag(id),
            _antiDiag(id)
        ];
        for (uint256 i; i < wins.length; i++) {
            uint256 win = _checkWin(wins[i]);
            if (win == X || win == O) return win;
        }
        return 0;
    }

    function _getSymbol(uint256 id, address player) public view returns (uint256) {
        if (player == games[id].playerX) return X;
        if (player == games[id].playerO) return O;
        return EMPTY;
    }

    function _validTurn(uint256 id) internal view returns (bool) {
        return currentTurn(id) == _getSymbol(id, msg.sender);
    }

    function _validPlayer(uint256 id) internal view returns (bool) {
        return msg.sender == games[id].playerX || msg.sender == games[id].playerO;
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

    function _row(uint256 id, uint256 row) internal view returns (uint256) {
        require(row < 3, "Invalid row");
        uint256 idx = 3 * row;
        return games[id].board[idx] * games[id].board[idx + 1] * games[id].board[idx + 2];
    }

    function _col(uint256 id, uint256 col) internal view returns (uint256) {
        require(col < 3, "Invalid column");
        return games[id].board[col] * games[id].board[col + 3] * games[id].board[col + 6];
    }

    function _diag(uint256 id) internal view returns (uint256) {
        return games[id].board[0] * games[id].board[4] * games[id].board[8];
    }

    function _antiDiag(uint256 id) internal view returns (uint256) {
        return games[id].board[2] * games[id].board[4] * games[id].board[6];
    }

    function _validTurn(uint256 id, uint256 symbol) internal view returns (bool) {
        return symbol == currentTurn(id);
    }

    function _emptySpace(uint256 id, uint256 i) internal view returns (bool) {
        return games[id].board[i] == EMPTY;
    }

    function _validSymbol(uint256 symbol) internal pure returns (bool) {
        return symbol == X || symbol == O;
    }
}
```

The tests:

```solidity
// SPDX-License-Identifier: Apache-2.0
pragma solidity 0.8.10;

import "ds-test/test.sol";
import "forge-std/Vm.sol";

import "../TicTacToken.sol";

contract User {

    TicTacToken internal ttt;
    Vm internal vm;
    address internal _address;

    constructor(address address_, TicTacToken _ttt, Vm _vm) {
        _address = address_;
        ttt = _ttt;
        vm = _vm;
    }

    function markSpace(uint256 id, uint256 space) public {
        vm.prank(_address);
        ttt.markSpace(id, space);
    }
}

contract TicTacTokenTest is DSTest {
    Vm internal vm = Vm(HEVM_ADDRESS);
    TicTacToken internal ttt;
    User internal playerX;
    User internal playerO;

    uint256 internal constant EMPTY = 0;
    uint256 internal constant X = 1;
    uint256 internal constant O = 2;

    address internal constant OWNER = address(1);
    address internal constant PLAYER_X = address(2);
    address internal constant PLAYER_O = address(3);

    function setUp() public {
        ttt = new TicTacToken(OWNER, PLAYER_X, PLAYER_O);
        playerX = new User(PLAYER_X, ttt, vm);
        playerO = new User(PLAYER_O, ttt, vm);
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
        uint256[9] memory actual = ttt.getBoard(0);

        for (uint256 i = 0; i < 9; i++) {
            assertEq(actual[i], expected[i]);
        }
    }

    function test_reset_board() public {
        playerX.markSpace(0, 3);
        playerO.markSpace(0, 0);
        playerX.markSpace(0, 4);
        playerO.markSpace(0, 1);
        playerX.markSpace(0, 5);
        vm.prank(OWNER);
        ttt.resetBoard(0);
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
        uint256[9] memory actual = ttt.getBoard(0);

        for (uint256 i = 0; i < 9; i++) {
            assertEq(actual[i], expected[i]);
        }
    }

    function test_can_mark_space_with_X() public {
        playerX.markSpace(0, 0);
        assertEq(ttt.getBoard(0)[0], X);
    }

    function test_can_mark_space_with_O() public {
        playerX.markSpace(0, 0);
        playerO.markSpace(0, 1);
        assertEq(ttt.getBoard(0)[1], O);
    }

    function test_cannot_overwrite_marked_space() public {
        playerX.markSpace(0, 0);

        vm.expectRevert("Already marked");
        playerO.markSpace(0, 0);
    }

    function test_symbols_must_alternate() public {
        playerX.markSpace(0, 0);
        vm.expectRevert("Not your turn");
        playerX.markSpace(0, 1);
    }

    function test_tracks_current_turn() public {
        assertEq(ttt.currentTurn(0), X);
        playerX.markSpace(0, 0);
        assertEq(ttt.currentTurn(0), O);
        playerO.markSpace(0, 1);
        assertEq(ttt.currentTurn(0), X);
    }

    function test_checks_for_horizontal_win() public {
        playerX.markSpace(0, 0);
        playerO.markSpace(0, 3);
        playerX.markSpace(0, 1);
        playerO.markSpace(0, 4);
        playerX.markSpace(0, 2);
        assertEq(ttt.winner(0), X);
    }

    function test_checks_for_horizontal_win_row2() public {
        playerX.markSpace(0, 3);
        playerO.markSpace(0, 0);
        playerX.markSpace(0, 4);
        playerO.markSpace(0, 1);
        playerX.markSpace(0, 5);
        assertEq(ttt.winner(0), X);
    }

    function test_checks_for_vertical_win() public {
        playerX.markSpace(0, 1);
        playerO.markSpace(0, 0);
        playerX.markSpace(0, 2);
        playerO.markSpace(0, 3);
        playerX.markSpace(0, 4);
        playerO.markSpace(0, 6);
        assertEq(ttt.winner(0), O);
    }

    function test_checks_for_diagonal_win() public {
        playerX.markSpace(0, 0);
        playerO.markSpace(0, 1);
        playerX.markSpace(0, 4);
        playerO.markSpace(0, 5);
        playerX.markSpace(0, 8);
        assertEq(ttt.winner(0), X);
    }

    function test_checks_for_antidiagonal_win() public {
        playerX.markSpace(0, 1);
        playerO.markSpace(0, 2);
        playerX.markSpace(0, 3);
        playerO.markSpace(0, 4);
        playerX.markSpace(0, 5);
        playerO.markSpace(0, 6);
        assertEq(ttt.winner(0), O);
    }

    function test_draw_returns_no_winner() public {
        playerX.markSpace(0, 4);
        playerO.markSpace(0, 0);
        playerX.markSpace(0, 1);
        playerO.markSpace(0, 7);
        playerX.markSpace(0, 2);
        playerO.markSpace(0, 6);
        playerX.markSpace(0, 8);
        playerO.markSpace(0, 5);
        assertEq(ttt.winner(0), 0);
    }

    function test_empty_board_returns_no_winner() public {
        assertEq(ttt.winner(0), 0);
    }

    function test_game_in_progress_returns_no_winner() public {
        playerX.markSpace(0, 1);
        assertEq(ttt.winner(0), 0);
    }

    function test_contract_owner() public {
        assertEq(ttt.owner(), OWNER);
    }

    function test_owner_can_reset_board() public {
        vm.prank(OWNER);
        ttt.resetBoard(0);
    }

    function test_non_owner_cannot_reset_board() public {
        vm.expectRevert("Unauthorized");
        ttt.resetBoard(0);
    }

    function test_stores_player_X() public {
        (address playerXAddr,,) = ttt.games(0);
        assertEq(playerXAddr, PLAYER_X);
    }

    function test_stores_player_O() public {
        (, address playerOAddr,) = ttt.games(0);
        assertEq(playerOAddr, PLAYER_O);
    }

    function test_auth_nonplayer_cannot_mark_space() public {
        vm.expectRevert("Unauthorized");
        ttt.markSpace(0, 0);
    }

    function test_auth_playerX_can_mark_space() public {
        vm.prank(PLAYER_X);
        ttt.markSpace(0, 0);
    }

    function test_auth_playerO_can_mark_space() public {
        vm.prank(PLAYER_X);
        ttt.markSpace(0, 0);

        vm.prank(PLAYER_O);
        ttt.markSpace(0, 1);
    }
}
```

Now that everything's parameterized by game ID, we have everything we need in place to support multiple simultaneous games.
