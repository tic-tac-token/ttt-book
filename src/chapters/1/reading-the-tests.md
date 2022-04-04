# Reading the tests
Foundry uses an [xUnit](https://en.wikipedia.org/wiki/XUnit) style test framework called `ds-test`, written in Solidity. Unlike other Solidity development frameworks, this makes it possible to write pure Solidity unit tests.

Test files are created as `.sol` files in the `src/test/` directory. It's a convention to put an extra `.t` in the test file name, so `Greeter.t.sol` is the test file for the `Greeter.sol` contract. (However, this is just a convention—the test runner will recognize as tests any files that inherit from `ds-test`).

Let's take a look at our test file, `Greeter.t.sol`. We haven't covered Solidity syntax in detail yet, but it should still be legible enough:

```solidity
// SPDX-License-Identifier: Apache-2.0
pragma solidity 0.8.10;

import "ds-test/test.sol";
import "forge-std/stdlib.sol";
import "forge-std/Vm.sol";

import "../Greeter.sol";

contract GreeterTest is DSTest {
    Vm public constant vm = Vm(HEVM_ADDRESS);

    Greeter internal greeter;

    function setUp() public {
        greeter = new Greeter("Hello");
    }

    function test_default_greeting() public {
       assertEq(greeter.greet(), "Hello, world!");
    }
    
    function test_custom_greeting() public {
       assertEq(greeter.greet("foundry"), "Hello, foundry!");
    }

    function test_get_greeting() public {
        assertEq(greeter.greeting(), "Hello");
    }
    
    function test_set_greeting() public {
        greeter.setGreeting("Ahoy-hoy");
        assertEq(greeter.greet(), "Ahoy-hoy, world!");
    }
    
    function test_non_owner_cannot_set_greeting() public {
        vm.prank(address(1));
        try greeter.setGreeting("Ahoy-hoy") {
            fail();
        } catch Error(string memory message) {
            assertEq(message, "Ownable: caller is not the owner");
        }
    }
}
```

Our test suite is defined as a Solidity _contract_, which looks a lot like a class or module in other languages. 

```solidity
contract GreeterTest is DSTest {
}
```

We create an instance of the contract we're testing in the `setUp` function, and access it later in our tests:

```solidity
    function setUp() public {
        greeter = new Greeter("Hello");
    }
```

Each test method is a public function prefixed with the word `test`. Inside each of these functions, we have access to assertions like `assertEq`: 

```solidity
    function test_default_greeting() public {
       assertEq(greeter.greet(), "Hello, world!");
    }
```

If you've used an xUnit style test framework before, this should all be pretty familiar. With the exception of a few keywords, this looks a lot like Javascript.
