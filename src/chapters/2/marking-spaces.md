# Marking a space

Let's put an "X" in the first space on the board, then read it back in our test. We'll pass the index of the space in our board array where we want to place it and a string to represent the "X" or "O" marker:

```solidity
    function test_can_mark_space_with_X() public {
        ttt.markSpace(0, "X");
        assertEq(ttt.board(0), "X");
    }
```

We can start with an empty `markSpace` function in the game contract:

```solidity
    function markSpace(uint256 space, string calldata symbol) public {}
```

> **Calldata**
>
> We can use `calldata` as the [data location](https://docs.soliditylang.org/en/latest/types.html#data-location) for `symbol` since it's a function argument.
Calldata is an immutable, ephemeral area where function arguments are stored. 

Here's the output from our failing test. We expected an "X" and instead got empty string:

```bash
Running 3 tests for src/test/TicTacToken.t.sol:TicTacTokenTest
[FAIL] test_can_mark_space_with_X() (gas: 20298)
Logs:
  Error: a == b not satisfied [string]
    Value a: 
    Value b: X

[PASS] test_get_board() (gas: 44215)
[PASS] test_has_empty_board() (gas: 46836)
Test result: FAILED. 2 passed; 1 failed; finished in 2.06ms

Failed tests:
[FAIL] test_can_mark_space_with_X() (gas: 20298)
```

This is what we expected, since we haven't implemented it yet. Again, we're getting back an empty string—the default value for a fixed length array of strings. Let's make it pass by setting the `space` index of the `board` to the `symbol` we pass in:

```solidity
    function markSpace(uint256 space, string calldata symbol) public {
        board[space] = symbol;
    }
```

All green again:

```bash
Running 3 tests for src/test/TicTacToken.t.sol:TicTacTokenTest
[PASS] test_can_mark_space_with_X() (gas: 31282)
[PASS] test_get_board() (gas: 44215)
[PASS] test_has_empty_board() (gas: 46836)
Test result: ok. 3 passed; 0 failed; finished in 1.90ms
```

We should now also be able to mark spaces with an "O". Let's try it out:

```solidity
    function test_can_mark_space_with_O() public {
        ttt.markSpace(0, "O");
        assertEq(ttt.board(0), "O");
    }
```

Since our function takes any string as an argument, we get this one for free:

```bash
Running 4 tests for src/test/TicTacToken.t.sol:TicTacTokenTest
[PASS] test_can_mark_space_with_O() (gas: 31337)
[PASS] test_can_mark_space_with_X() (gas: 31305)
[PASS] test_get_board() (gas: 44171)
[PASS] test_has_empty_board() (gas: 46859)
Test result: ok. 4 passed; 0 failed; finished in 1.90ms
```

Of course, we get a lot more for free, too! The caller can mark the board with any arbitrary string: a "B", a "💖", a "Solidity rocks!", whatever. Let's add some validation to allow only the markers "X" and "O". 
