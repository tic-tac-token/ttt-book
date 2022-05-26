# Validating markers

We should be able to mark a square on the board with an "X" or "O" symbol, but not any other. We'd like our `markSpace` function to throw an error if it's called with an invalid symbol. 

There are several expressions for [error handling in Solidity](https://docs.soliditylang.org/en/latest/control-structures.html#error-handling-assert-require-revert-and-exceptions). The most common are the `require` function with a string error message and using the `revert` statement with a custom error. Let's take a quick look at both.

## Using `require`
The `require` function takes a condition to check and an error message to return if the condition is false. Think of it as checking some assertion about the state of our code that must always be true. For example:

```solidity
require(cats >= 100, "Must have more than 100 cats");
require(block.timestamp < expiration, "Current time is past expiration date");
require(token.ownerOf(id) == msg.sender, "Caller must be token owner");
require(a + b + c == 11, "Sum must equal eleven");
```

If the condition in a `require` function is false, it reverts and returns an error that includes the provided error message. The `require` function is nice and concise, and it remains the most widely used idiom for throwing errors in Solidity.    

## Using `revert` with custom errors 

Solidity versions since v0.8.4 support _custom errors_, which are more verbose than `require` but have a few new capabilities.

Here are the same errors as above defined as custom errors:

```solidity
error NotEnoughCats(uint256 numCats);
if (cats < 100) {
    revert NotEnoughCats(cats);
}

error Expired();
if (block.timestamp >= expiration) {
    revert Expired();
}

error UnauthorizedCaller();
if (msg.sender !== token.ownerOf(id)) {
    revert UnauthorizedCaller();
}

error SumIsNotEleven(uint256 a, uint256 b, uint256 c);
if (a + b + c !== 11) {
    revert SumIsNotEleven(a, b, c);
}
```

First, we define a custom error using the `error` keyword, then use the `revert` keyword to throw an exception under certain conditions. Note that we've reversed the logic in each case: `require` is used to ensure some condition is true, while custom errors are usually thrown when some condition is false.

One advantage of custom errors over `require` strings is that they can take parameters, like the `uint256` parameters to `NotEnoughCats` and `SumIsNotEleven` above. This is a good way to return structured data with your error that can be more descriptive than a simple require string.

> **What does it mean to `revert`?**
>
> Ethereum transactions are atomic, like database transactions. When a transaction reverts, execution stops, and the EVM rolls back any state changes and side effects of the reverted call, including changes in external contracts.

## Testing errors
To test errors, we'll use a new Forge feature: the `vm.expectRevert` cheatcode. To access cheatcodes, we need to import the `Vm.sol` helper from [forge-std](https://github.com/brockelmore/forge-std) and set it up in our tests: 

```solidity
// SPDX-License-Identifier: Apache-2.0
pragma solidity 0.8.10;

import "ds-test/test.sol";
import "forge-std/Vm.sol";

import "../TicTacToken.sol";

contract TicTacTokenTest is DSTest {
    Vm internal vm = Vm(HEVM_ADDRESS);
    TicTacToken internal ttt;

    function setUp() public {
        ttt = new TicTacToken();
    }
    
    function test_cannot_mark_space_with_Z() public {
        vm.expectRevert("Invalid symbol");
        ttt.markSpace(0, "Z");
    }
}
```

The `vm.expectRevert` cheatcode expects the error string for `require` statements as its argument.

> **Forge Standard Library**
>
> The Forge Standard Library, or [forge-std](https://github.com/brockelmore/forge-std) is a collection of helper contracts for use with Foundry. Among these is `Vm.sol`, a helper interface to Foundry cheatcodes.
>
> To install `forge-std`, run `forge install brockelmore/forge-std`

Let's give this test a try:

```bash
$ forge test
[⠊] Compiling...
[⠒] Compiling 2 files with 0.8.10
[⠢] Solc finished in 9.84ms
Error: 
   0: Compiler run failed
      TypeError: Operator == not compatible with types string calldata and literal_string "X"
        --> /Users/ecm/Projects/ttt-book-code/src/TicTacToken.sol:12:17:
         |
      12 |         require(symbol == "X" || symbol == "O");
         |                 ^^^^^^^^^^^^^

      

      TypeError: Operator == not compatible with types string calldata and literal_string "O"
        --> /Users/ecm/Projects/ttt-book-code/src/TicTacToken.sol:12:34:
         |
      12 |         require(symbol == "X" || symbol == "O");
         |                                  ^^^^^^^^^^^^^
```

Hmm, we can't simply use the `==` operator to compare two strings. We'll need to do it some other way. 

One common idiom to compare strings is to use a hash comparison, by converting the strings to bytes and using the built in `keccak256` function. Let's create a `_compareStrings` helper and use it to do the comparison:

```solidity
pragma solidity 0.8.10;

contract TicTacToken {
    string[9] public board;

    function getBoard() public view returns (string[9] memory) {
        return board;
    }

    function markSpace(uint256 space, string calldata symbol) public {
        require(_compareStrings(symbol, "X") || _compareStrings(symbol, "O"), "Invalid symbol");
        board[space] = symbol;
    }

    function _compareStrings(string memory a, string memory b) internal pure returns (bool) {
        return keccak256(abi.encodePacked(a)) == keccak256(abi.encodePacked(b));
    }
}
```

Note that this function is `internal` since it's an internal helper. It's `pure` since it doesn't read or write any state. And we've prefixed it with an underscore, which is the preferred style for internal and private functions.

Let's see if our `vm.expectRevert` assertion passes now:

```bash
$ forge test
[⠊] Compiling...
[⠆] Compiling 2 files with 0.8.10
[⠰] Solc finished in 179.19ms
Compiler run successful

Running 5 tests for src/test/TicTacToken.t.sol:TicTacTokenTest
[PASS] test_can_mark_space_with_O() (gas: 33252)
[PASS] test_can_mark_space_with_X() (gas: 32280)
[PASS] test_cannot_mark_space_with_Z() (gas: 12720)
[PASS] test_get_board() (gas: 44689)
[PASS] test_has_empty_board() (gas: 47219)
Test result: ok. 5 passed; 0 failed; finished in 2.44ms
```

It works! Before we move on, let's refactor to make this a little more readable by extracting another helper method:

```solidity
// SPDX-License-Identifier: Apache-2.0
pragma solidity 0.8.10;

contract TicTacToken {
    string[9] public board;

    function getBoard() public view returns (string[9] memory) {
        return board;
    }

    function markSpace(uint256 space, string calldata symbol) public {
        require(_validSymbol(symbol), "Invalid symbol");
        board[space] = symbol;
    }

    function _validSymbol(string calldata symbol) internal pure returns (bool) {
        return _compareStrings(symbol, "X") || _compareStrings(symbol, "O");
    }

    function _compareStrings(string memory a, string memory b) internal pure returns (bool) {
        return keccak256(abi.encodePacked(a)) == keccak256(abi.encodePacked(b));
    }
}
```
