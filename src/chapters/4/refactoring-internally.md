# Refactoring internally

Let's refactor the way we refer to game data internally to read from the `games` mapping rather than global variables. First, let's define our new `Game` struct and add a mapping:

```solidity
contract TicTacToken {
    uint256[9] public board;
    address public owner;
    address public playerX;
    address public playerO;

    struct Game {
        address playerX;
        address playerO;
        uint256 turns;
        uint256[9] board;
    }
    mapping(uint256 => Game) games;

    uint256 internal constant EMPTY = 0;
    uint256 internal constant X = 1;
    uint256 internal constant O = 2;
    uint256 internal _turns;

    constructor(address _owner, address _playerX, address _playerO) {
        owner = _owner;
        playerX = _playerX;
        playerO = _playerO;
    }
}
```

Since nothing is using either of these yet, all our tests still pass:

```bash
$ forge test
Running 25 tests for src/test/TicTacToken.t.sol:TicTacTokenTest
[PASS] test_auth_nonplayer_cannot_mark_space() (gas: 14913)
[PASS] test_auth_playerO_can_mark_space() (gas: 84612)
[PASS] test_auth_playerX_can_mark_space() (gas: 57687)
[PASS] test_can_mark_space_with_O() (gas: 106093)
[PASS] test_can_mark_space_with_X() (gas: 67849)
[PASS] test_cannot_overwrite_marked_space() (gas: 83135)
[PASS] test_checks_for_antidiagonal_win() (gas: 222815)
[PASS] test_checks_for_diagonal_win() (gas: 198684)
[PASS] test_checks_for_horizontal_win() (gas: 196829)
[PASS] test_checks_for_horizontal_win_row2() (gas: 197126)
[PASS] test_checks_for_vertical_win() (gas: 221544)
[PASS] test_contract_owner() (gas: 7661)
[PASS] test_draw_returns_no_winner() (gas: 269962)
[PASS] test_empty_board_returns_no_winner() (gas: 32303)
[PASS] test_game_in_progress_returns_no_winner() (gas: 92559)
[PASS] test_get_board() (gas: 29218)
[PASS] test_has_empty_board() (gas: 32173)
[PASS] test_msg_sender() (gas: 238582)
[PASS] test_non_owner_cannot_reset_board() (gas: 12706)
[PASS] test_owner_can_reset_board() (gas: 32939)
[PASS] test_reset_board() (gas: 158387)
[PASS] test_stores_player_O() (gas: 7749)
[PASS] test_stores_player_X() (gas: 7794)
[PASS] test_symbols_must_alternate() (gas: 70209)
[PASS] test_tracks_current_turn() (gas: 108163)
Test result: ok. 25 passed; 0 failed; finished in 2.48ms
```

Now for a big refactor: let's move the game state from the global state variables to the first struct in the `games` mapping. That means anywhere we're referring to `board`, `playerX`, `playerO`, or `_turns`, we should instead point to `games[0].board`, `games[0].playerX`, etc. (Find-and-replace can help).

Here's our full contract after this change:

```solidity
contract TicTacToken {
    uint256[9] public board;
    address public owner;
    address public playerX;
    address public playerO;

    struct Game {
        address playerX;
        address playerO;
        uint256 turns;
        uint256[9] board;
    }
    mapping(uint256 => Game) games;

    uint256 internal constant EMPTY = 0;
    uint256 internal constant X = 1;
    uint256 internal constant O = 2;
    uint256 internal _turns;

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

Was it a clean refactor? Let's run the tests and see:

```bash
$ forge test
Running 25 tests for src/test/TicTacToken.t.sol:TicTacTokenTest
[PASS] test_auth_nonplayer_cannot_mark_space() (gas: 14946)
[PASS] test_auth_playerO_can_mark_space() (gas: 85032)
[PASS] test_auth_playerX_can_mark_space() (gas: 57870)
[PASS] test_cannot_overwrite_marked_space() (gas: 83453)
[PASS] test_checks_for_antidiagonal_win() (gas: 225083)
[PASS] test_checks_for_diagonal_win() (gas: 200715)
[PASS] test_checks_for_horizontal_win() (gas: 198860)
[PASS] test_checks_for_horizontal_win_row2() (gas: 199157)
[PASS] test_checks_for_vertical_win() (gas: 223812)
[PASS] test_contract_owner() (gas: 7661)
[PASS] test_draw_returns_no_winner() (gas: 272650)
[PASS] test_empty_board_returns_no_winner() (gas: 33311)
[PASS] test_game_in_progress_returns_no_winner() (gas: 93750)
[PASS] test_get_board() (gas: 29269)
[PASS] test_has_empty_board() (gas: 32173)
[PASS] test_msg_sender() (gas: 238582)
[PASS] test_non_owner_cannot_reset_board() (gas: 12706)
[PASS] test_owner_can_reset_board() (gas: 32996)
[PASS] test_reset_board() (gas: 159292)
[FAIL] test_stores_player_O() (gas: 18614)
Logs:
  Error: a == b not satisfied [address]
    Expected: 0x0000000000000000000000000000000000000003
      Actual: 0x0000000000000000000000000000000000000000

[FAIL] test_stores_player_X() (gas: 18659)
Logs:
  Error: a == b not satisfied [address]
    Expected: 0x0000000000000000000000000000000000000002
      Actual: 0x0000000000000000000000000000000000000000

[FAIL] test_can_mark_space_with_O() (gas: 119317)
Logs:
  Error: a == b not satisfied [uint]
    Expected: 2
      Actual: 0

[FAIL] test_can_mark_space_with_X() (gas: 80836)
Logs:
  Error: a == b not satisfied [uint]
    Expected: 1
      Actual: 0

[PASS] test_symbols_must_alternate() (gas: 70440)
[PASS] test_tracks_current_turn() (gas: 108637)
Test result: FAILED. 21 passed; 4 failed; finished in 4.08ms

Failed tests:
[FAIL] test_can_mark_space_with_O() (gas: 119317)
[FAIL] test_can_mark_space_with_X() (gas: 80836)
[FAIL] test_stores_player_O() (gas: 18614)
[FAIL] test_stores_player_X() (gas: 18659)

Encountered a total of 4 failing tests, 21 tests succeeded
```

Pretty good, but we've missed a few spots where our tests are relying on default getters for `board`, `playerO`, and `playerX`:

```solidity
    function test_stores_player_X() public {
        assertEq(ttt.playerX(), PLAYER_X);
    }

    function test_stores_player_O() public {
        assertEq(ttt.playerO(), PLAYER_O);
    }

    function test_can_mark_space_with_X() public {
        playerX.markSpace(0);
        assertEq(ttt.board(0), X);
    }

    function test_can_mark_space_with_O() public {
        playerX.markSpace(0);
        playerO.markSpace(1);
        assertEq(ttt.board(1), O);
    }
```

Let's update these to read from the mapping instead. Note that we can use multiple assignment to access the values of a struct:

```solidity
    function test_stores_player_X() public {
        (address playerXAddr,,) = ttt.games(0);
        assertEq(playerXAddr, PLAYER_X);
    }

    function test_stores_player_O() public {
        (, address playerOAddr,) = ttt.games(0);
        assertEq(playerOAddr, PLAYER_O);
    }

    function test_can_mark_space_with_X() public {
        playerX.markSpace(0);
        assertEq(ttt.getBoard()[0], X);
    }

    function test_can_mark_space_with_O() public {
        playerX.markSpace(0);
        playerO.markSpace(1);
        assertEq(ttt.getBoard()[1], O);
    }
```

Confirm that our tests pass:

```bash
$ forge test
Running 25 tests for src/test/TicTacToken.t.sol:TicTacTokenTest
[PASS] test_auth_nonplayer_cannot_mark_space() (gas: 14946)
[PASS] test_auth_playerO_can_mark_space() (gas: 85032)
[PASS] test_auth_playerX_can_mark_space() (gas: 57870)
[PASS] test_can_mark_space_with_O() (gas: 123671)
[PASS] test_can_mark_space_with_X() (gas: 87187)
[PASS] test_cannot_overwrite_marked_space() (gas: 83453)
[PASS] test_checks_for_antidiagonal_win() (gas: 225094)
[PASS] test_checks_for_diagonal_win() (gas: 200726)
[PASS] test_checks_for_horizontal_win() (gas: 198871)
[PASS] test_checks_for_horizontal_win_row2() (gas: 199168)
[PASS] test_checks_for_vertical_win() (gas: 223823)
[PASS] test_contract_owner() (gas: 7696)
[PASS] test_draw_returns_no_winner() (gas: 272661)
[PASS] test_empty_board_returns_no_winner() (gas: 33322)
[PASS] test_game_in_progress_returns_no_winner() (gas: 93761)
[PASS] test_get_board() (gas: 29291)
[PASS] test_has_empty_board() (gas: 32470)
[PASS] test_msg_sender() (gas: 238687)
[PASS] test_non_owner_cannot_reset_board() (gas: 12728)
[PASS] test_owner_can_reset_board() (gas: 33018)
[PASS] test_reset_board() (gas: 159327)
[PASS] test_stores_player_O() (gas: 12233)
[PASS] test_stores_player_X() (gas: 12299)
[PASS] test_symbols_must_alternate() (gas: 70440)
[PASS] test_tracks_current_turn() (gas: 108670)
Test result: ok. 25 passed; 0 failed; finished in 4.00ms
```

Finally, we can remove the global `board`, `playerX`, `playerO`, and `_turns`:

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
```

Up next, updating the external interface to take game ID as a parameter.
