# Test driving the board

Let's create our first test. To start, let's represent the empty Tic Tac Toe board as an array of strings, and check that each element in the array is the empty string: 

```solidity
// SPDX-License-Identifier: Apache-2.0
pragma solidity 0.8.10;

import "ds-test/test.sol";

import "../TicTacToken.sol";

contract TicTacTokenTest is DSTest {
    TicTacToken internal ttt;

    function setUp() public {
        ttt = new TicTacToken();
    }

    function test_has_empty_board() public {
        for (uint256 i=0; i<9; i++) {
            assertEq(ttt.board(i), "");
        }
    }
}
```

> **Automatic getters**
>
> Notice the `ttt.board(i)` function call inside the loop, which takes an integer index as its argument? Solidity automatically creates [getter functions](https://docs.soliditylang.org/en/latest/contracts.html#getter-functions) for public state variables. These getter functions behave differently depending on the type of the variable. If your public state variable is a primitive type, like `uint8`, `address`, or `bool`, its getter returns its value directly. If it's a more complex type like an array or mapping, its getter function takes an argument: the index of an item to retrieve from the array or the key to access in the mapping. In the test above, we retrieve each element in the array by index. 

Let's run this test and see where the compiler points us:

```bash
$ forge test
[⠊] Compiling...
[⠒] Compiling 1 files with 0.8.10
[⠢] Solc finished in 8.50ms
Error: 
   0: Compiler run failed
      TypeError: Member "board" not found or not visible after argument-dependent lookup in contract TicTacToken.
        --> /Users/ecm/Projects/ttt-book-code/src/test/TicTacToken.t.sol:17:22:
         |
      17 |             assertEq(ttt.board(i), "");
         |                      ^^^^^^^^^
```

Great—we need to add something named `board` to our contract in order to call this function. 

We know our board has 9 spaces, so we can declare a [fixed size array](https://docs.soliditylang.org/en/latest/types.html#arrays). We'll make it `public`, which will generate the `board(i)` getter:

```solidity
// SPDX-License-Identifier: Apache-2.0
pragma solidity 0.8.10;

contract TicTacToken {
    string[9] public board;
}
```

Run our tests...

```bash
$ forge test
[⠊] Compiling...
[⠢] Compiling 2 files with 0.8.10
[⠆] Solc finished in 82.77ms
Compiler run successful

Running 1 test for src/test/TicTacToken.t.sol:TicTacTokenTest
[PASS] test_has_empty_board() (gas: 46121)
Test result: ok. 1 passed; 0 failed; finished in 1.46ms
```

They pass!

> **Default values**
>
> Wait, what…they pass?! This might come as a surprise, since all we've done is create a state variable that should be an empty array. We never inserted any empty strings, but somehow our test passed! There is no concept of "undefined," or "null" in Solidity. Instead, newly declared values always have a default value that depends on their type. A `bool` will be false, a `uint256` will be 0, and a string will be `""`. A fixed size array like our `string[9]` will have each of its elements initialized to the default value of its type. In our case, that's 9 empty strings. See the [Solidity docs](https://docs.soliditylang.org/en/latest/control-structures.html#scoping-and-declarations) for more details on this behavior.

OK, on to another test. Working with only our default getter is prety awkward, since we can only access individual items by index. How about a function that returns the whole board as an array?

```solidity
// SPDX-License-Identifier: Apache-2.0
pragma solidity 0.8.10;

import "ds-test/test.sol";

import "../TicTacToken.sol";

contract TicTacTokenTest is DSTest {
    TicTacToken internal ttt;

    function setUp() public {
        ttt = new TicTacToken();
    }

    function test_has_empty_board() public {
        for (uint256 i=0; i<9; i++) {
            assertEq(ttt.board(i), "");
        }
    }
    function test_get_board() public {
        string[9] memory expected = ["", "", "", "", "", "", "", "", ""];
        string[9] memory actual = ttt.getBoard();

        for (uint256 i=0; i<9; i++) {
            assertEq(actual[i], expected[i]);
        }
    }
}
```

We still need to iterate over each element in our test because `ds-test` doesn't include an array equality matcher. We'll define a new `getBoard` function that returns the whole `string[9] memory` representing our board.

Let's run the tests to make sure we see the expected failure:

```bash
$ forge test
[⠊] Compiling...
[⠢] Compiling 1 files with 0.8.10
[⠆] Solc finished in 9.15ms
Error: 
   0: Compiler run failed
      TypeError: Member "getBoard" not found or not visible after argument-dependent lookup in contract TicTacToken.
        --> /Users/ecm/Projects/ttt-book-code/src/test/TicTacToken.t.sol:22:35:
         |
      22 |         string[9] memory actual = ttt.getBoard();
         |                                   ^^^^^^^^^^^^
```

And add a `getBoard` function:

```solidity
// SPDX-License-Identifier: Apache-2.0
pragma solidity 0.8.10;

contract TicTacToken {
    string[9] public board;

    function getBoard() public view returns (string[9] memory) {
        return board;
    }
}
```
> **Views**
>
> Notice that we can declare this function as a `view` because it's read only and doesn't modify any state. See the Solidity docs on [state mutability](https://docs.soliditylang.org/en/latest/contracts.html#state-mutability) for more.

Now we can run our tests and see them pass:

```bash
$ forge test
[⠊] Compiling...
[⠆] Compiling 2 files with 0.8.10
[⠰] Solc finished in 130.38ms
Compiler run successful

Running 2 tests for src/test/TicTacToken.t.sol:TicTacTokenTest
[PASS] test_get_board() (gas: 44193)
[PASS] test_has_empty_board() (gas: 46814)
Test result: ok. 2 passed; 0 failed; finished in 2.16ms
```

On to some more interesting behavior.

Now we can run our tests and see them pass:

```bash
$ forge test
[⠊] Compiling...
[⠆] Compiling 2 files with 0.8.10
[⠰] Solc finished in 130.38ms
Compiler run successful

Running 2 tests for src/test/TicTacToken.t.sol:TicTacTokenTest
[PASS] test_get_board() (gas: 44193)
[PASS] test_has_empty_board() (gas: 46814)
Test result: ok. 2 passed; 0 failed; finished in 2.16ms
```

Great, we can get our board back as an array. On to some more interesting behavior. How about marking a space with a symbol?

```solidity
    function test_can_mark_space_with_X() public {
        ttt.markSpace(0, "X");
        assertEq(ttt.board(0), "X");
    }
```

We'll start with an empty function in the game contract:

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

Let's make it pass by setting the `space` index of the `board` to the `symbol` we pass in:

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
