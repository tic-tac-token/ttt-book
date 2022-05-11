# Creating a token

Now that we've explored how ERC20 tokens work, you may have noticed some similiarities with the leaderboard in our game. The internal mapping storing the number of points awarded by player address in our game is not too different from the internal mapping storing token balances in an ERC20. Let's lean into this similarity and use a token as our mechanism for awarding points.

Of course, tokens are transferrable, so this is conceptually a little bit different than our existing points scoreboard: less like the high score screen at the end of an arcade game, and more like a skee-ball machine that gives out prize tickets.

To start, we need a token of our own. Let's create a new, empty test file: `src/test/Token.t.sol`:

```solidity
// SPDX-License-Identifier: Apache-2.0
pragma solidity 0.8.10;

import "ds-test/test.sol";
import "forge-std/Vm.sol";

import "../Token.sol";

contract TicTacTokenTest is DSTest {
    Vm internal vm = Vm(HEVM_ADDRESS);

    Token internal token;

    function setUp() public {
        token = new Token();
    }
}
```

...and a new, empty contract for our token, `src/Token.sol`:

```solidity
// SPDX-License-Identifier: Apache-2.0
pragma solidity 0.8.10;

contract Token {}
```

We'll add a few inital tests for the token metadata attributes: name, symbol, and decimals:

```solidity
    function test_token_name() public {
        assertEq(token.name(), "Tic Tac Token");
    }

    function test_token_symbol() public {
        assertEq(token.symbol(), "TTT");
    }

    function test_token_decimals() public {
        assertEq(token.decimals(), 18);
    }
```

Over in our token contract, let's import and use OpenZeppelin ERC20:

```solidity
import "openzeppelin-contracts/contracts/token/ERC20.sol";

contract Token is ERC20 {}
```

Run the tests, and we'll see a compiler error:

```bash
$ forge test
[⠊] Compiling...
[⠒] Compiling 2 files with 0.8.10
[⠢] Solc finished in 12.85ms
Error:
   0: Compiler run failed
      TypeError: Contract "Token" should be marked as abstract.
       --> /Users/ecm/Projects/ttt-book-code/src/Token.sol:6:1:
        |
      6 | contract Token is ERC20 {}
        | ^^^^^^^^^^^^^^^^^^^^^^^^^^
      Note: Missing implementation:
        --> /Users/ecm/Projects/ttt-book-code/lib/openzeppelin-contracts/contracts/token/ERC20/ERC20.sol:54:5:
         |
      54 |     constructor(string memory name_, string memory symbol_) {
         |     ^ (Relevant source part starts here and spans across multiple lines).
```

This Solidity error is a little cryptic, but it's indicating that our base contract `ERC20` is expecting constructor arguments. In this case, we need to pass the token name and symbol at construction time.

Let's add a `constructor()` function and pass the name and symbol as arguments. We can use a [modifier-like syntax](https://docs.soliditylang.org/en/v0.8.13/contracts.html#arguments-for-base-constructors) inline next to the constructor function to pass the arguments needed by `ERC20`:

```solidity
import "openzeppelin-contracts/contracts/token/ERC20/ERC20.sol";

contract Token is ERC20 {

    constructor() ERC20("Tic Tac Token", "TTT") {}

}
```

With our base contract set up correctly, our tests should pass:

```bash
$ forge test
Running 3 tests for src/test/Token.t.sol:TicTacTokenTest
[PASS] test_token_decimals() (gas: 5481)
[PASS] test_token_name() (gas: 9688)
[PASS] test_token_symbol() (gas: 9687)
Test result: ok. 3 passed; 0 failed; finished in 1.67ms
```

In order to issue tokens to winners, we'll need to expose the ability to mint new tokens. OpenZeppelin ERC20 provides an internal `_mint` [function](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_mint-address-uint256-) we can use for this purpose.

Let's start with a test. We'll add a public `mint` function to our contract that takes an address to credit with the newly created tokens and an amount of tokens to mint. In our test, we'll mint 100 tokens to the test contract, then call `balanceOf` on the token contract to check that we've received them:

```solidity
    function test_mint_to_user() public {
        token.mint(address(this), 100 ether);
        assertEq(token.balanceOf(address(this)), 100 ether);
    }
```

> **Ether units**
>
> The keyword `ether` in the example above may look a little strange. `ether` is a [globally available unit](https://docs.soliditylang.org/en/latest/units-and-global-variables.html#ether-units) in Solidity that can apply to any numeric literal. `1 ether` is equivalent to `1e18`. This is the number of decimal places that denominate ETH, and the number of decimal places that denominate most ERC20 tokens, including ours in the example above. Using the `100 ether` in the example above is shorthand for `100 * 1e18`, or "100 tokens".
>
> The units `gwei` and `wei` are also available keywords representing smaller denominations.

Let's add a public function that will call the internal `_mint` function from our base ERC20 contract. Just to make sure our tests are working, we'll leave it empty for now:

```solidity
    function mint(address account, uint256 amount) public {
    }
```

Run the tests to verify:

```
$ forge test
Running 4 tests for src/test/Token.t.sol:TicTacTokenTest
[FAIL] test_mint_to_user() (gas: 19258)
Logs:
  Error: a == b not satisfied [uint]
    Expected: 100000000000000000000
      Actual: 0

[PASS] test_token_decimals() (gas: 5504)
[PASS] test_token_name() (gas: 9644)
[PASS] test_token_symbol() (gas: 9710)
Test result: FAILED. 3 passed; 1 failed; finished in 539.42µs

Failed tests:
[FAIL] test_mint_to_user() (gas: 19258)

Encountered a total of 1 failing tests, 3 tests succeeded
```

Great! This is the failure we should expect: our account had a balance of zero tokens, but we expected 100. (Note the 18 decimal places in the expected amount!)

Let's update the public mint function to actually call the internal `_mint` function and pass through the arguments:

```solidity
    function mint(address account, uint256 amount) public {
        _mint(account, amount);
    }
```

When we run the tests again, we'll see that our account has the expected balance:

```bash
$ forge test
Running 4 tests for src/test/Token.t.sol:TicTacTokenTest
[PASS] test_mint_to_user() (gas: 52968)
[PASS] test_token_decimals() (gas: 5504)
[PASS] test_token_name() (gas: 9644)
[PASS] test_token_symbol() (gas: 9710)
Test result: ok. 4 passed; 0 failed; finished in 701.88µs
```

We now have a fully functional ERC20. It's not much use testing the built in behavior of our parent contract, but here's just one more test to prove the point. We'll transfer some of our tokens to another address and check the balances of both:

```solidity
    function test_transfer_tokens() public {
        token.mint(address(this), 100 ether);
        token.transfer(address(42), 50 ether);

        assertEq(token.balanceOf(address(this)), 50 ether);
        assertEq(token.balanceOf(address(42)), 50 ether);
    }
```

Run our tests, and this should pass, since we've inherited this behavior from the base ERC20 contract.

```bash
$ forge test

Running 5 tests for src/test/Token.t.sol:TicTacTokenTest
[PASS] test_mint_to_user() (gas: 52968)
[PASS] test_token_decimals() (gas: 5504)
[PASS] test_token_name() (gas: 9644)
[PASS] test_token_symbol() (gas: 9710)
[PASS] test_transfer_tokens() (gas: 79779)
Test result: ok. 5 passed; 0 failed; finished in 701.58µs
```

We're missing one more important thing: permissioning the `mint` function so that. Let's use OpenZeppelin `Ownable` to limit access to `mint` with the `onlyOwner` modifier. If we `prank` another address and try to mint, the call should revert:

```solidity
    function test_non_owner_cannot_mint() public {
        vm.prank(address(42));
        vm.expectRevert("Ownable: caller is not the owner");
        token.mint(address(this), 100 ether);
    }
```

We can import `Ownable` like we did with `ERC20`, inherit it and use the `onlyOwner` modifier. Note that we don't need to pass any constructor arguments this time:

```solidity
import "openzeppelin-contracts/contracts/token/ERC20/ERC20.sol";

contract Token is ERC20, Ownable {

    constructor() ERC20("Tic Tac Token", "TTT") {}

    function mint(address account, uint256 amount) public onlyOwner {
        _mint(account, amount);
    }

}
```

With the modifier in place, our test should pass:

```bash
$ forge test

Running 6 tests for src/test/Token.t.sol:TicTacTokenTest
[PASS] test_mint_to_user() (gas: 55220)
[PASS] test_non_owner_cannot_mint() (gas: 13342)
[PASS] test_token_decimals() (gas: 5460)
[PASS] test_token_name() (gas: 9665)
[PASS] test_token_symbol() (gas: 9753)
[PASS] test_transfer_tokens() (gas: 81899)
Test result: ok. 6 passed; 0 failed; finished in 828.25µs
```
