# Property based testing
We could leave our fully-functional Fizzbuzz here, but before we wrap up, let's take a look at another powerful features of the Forge test runner: property-based "fuzz tests."

The four unit tests we wrote to test drive our solution touch only a few points in the enormous domain of our `fizzbuzz` function. Sure, we picked them because they are specific, special cases, but wouldn't it be nice to have a little more confidence by testing a few more? We're in luck, because Forge makes it easy to turn our unit tests into _property-based tests_.

Forge will interpret any unit test function that takes an argument as a property based test, and run it multiple times with randomly assigned values. Let's just update our first unit test for now, by adding a `uint256` argument to the test function and replacing the hardcoded value in the assertion.

Here's our existing unit test:

```solidity
    function test_returns_fizz_when_divisible_by_three() public {
        assertEq(fizzbuzz.fizzbuzz(3), "fizz");
        assertEq(fizzbuzz.fizzbuzz(6), "fizz");
        assertEq(fizzbuzz.fizzbuzz(27), "fizz");
    }
```

Here it is as a property based test parameterized by an integer `n`:

```solidity
    function test_returns_fizz_when_divisible_by_three(uint256 n) public {
        assertEq(fizzbuzz.fizzbuzz(n), "fizz");
    }
```

What happens when we run this test?

```bash
$ forge test
[⠊] Compiling...
No files changed, compilation skipped

Running 4 tests for src/test/FizzBuzz.t.sol:FizzBuzzTest
[PASS] test_returns_buzz_when_divisible_by_five() (gas: 12478)
[FAIL. Counterexample: calldata=0x3f56f23b4000000000000000000000000000000000000000000000000000000000000000, args=[28948022309329048855892746252171976963317496166410141009864396001978282409984]] test_returns_fizz_when_divisible_by_three(uint256) (runs: 1, μ: 7363, ~: 7363)
Logs:
  Error: a == b not satisfied [string]
    Value a: 28948022309329048855892746252171976963317496166410141009864396001978282409984
    Value b: fizz

[PASS] test_returns_fizzbuzz_when_divisible_by_three_and_five() (gas: 12210)
[PASS] test_returns_number_as_string_otherwise() (gas: 8039)
Test result: FAILED. 3 passed; 1 failed; finished in 49.08ms

Failed tests:
[FAIL. Counterexample: calldata=0x3f56f23b4000000000000000000000000000000000000000000000000000000000000000, args=[28948022309329048855892746252171976963317496166410141009864396001978282409984]] test_returns_fizz_when_divisible_by_three(uint256) (runs: 1, μ: 7363, ~: 7363)

Encountered a total of 1 failing tests, 3 tests succeeded
```

They failed! Can you see why?

It's a little hard to tell with the massive integer we got back as our counterexample, but we need to remember to limit our input parameter `n` to specific values that are divisible by three. The pattern for doing this in Forge is to use a special cheatcode, `vm.assume(bool)`. This will filter out any fuzz test inputs that don't match the specified condition. We'll need to set up access to the `Vm` interface and add a line to our test, like this:

```solidity
import "ds-test/test.sol";
import "forge-std/Vm.sol";
import "../FizzBuzz.sol";

contract FizzBuzzTest is DSTest {
    Vm public constant vm = Vm(HEVM_ADDRESS);
    FizzBuzz internal fizzbuzz;

    function setUp() public {
        fizzbuzz = new FizzBuzz();
    }

    function test_returns_fizz_when_divisible_by_three(uint256 n) public {
        vm.assume(n % 3 == 0);
        vm.assume(n % 5 != 0);
        assertEq(fizzbuzz.fizzbuzz(n), "fizz");
    }
}
```

In this case we're filtering out anything that's not divisible by 3 (it shouldn't return "fizz") and anything that's also divisible by 5 (it should return "fizzbuzz", not "fizz"). Let's try again:

```bash
$ forge test
Running 4 tests for src/test/FizzBuzz.t.sol:FizzBuzzTest
[PASS] test_returns_buzz_when_divisible_by_five() (gas: 12478)
[PASS] test_returns_fizz_when_divisible_by_three(uint256) (runs: 256, μ: 10746, ~: 10746)
[PASS] test_returns_fizzbuzz_when_divisible_by_three_and_five() (gas: 12210)
[PASS] test_returns_number_as_string_otherwise() (gas: 8039)
Test result: ok. 4 passed; 0 failed; finished in 38.66ms
```

It works, and we've turned our single unit test into 256 assertions!

Here's how we can update all the tests to run as property-based tests:

```solidity
// SPDX-License-Identifier: Apache-2.0
pragma solidity 0.8.10;

import "ds-test/test.sol";
import "forge-std/Vm.sol";
import "openzeppelin-contracts/contracts/utils/Strings.sol";

import "../FizzBuzz.sol";

contract FizzBuzzTest is DSTest {
    Vm public constant vm = Vm(HEVM_ADDRESS);
    FizzBuzz internal fizzbuzz;

    function setUp() public {
        fizzbuzz = new FizzBuzz();
    }

    function test_returns_fizz_when_divisible_by_three(uint256 n) public {
        vm.assume(n % 3 == 0);
        vm.assume(n % 5 != 0);
        assertEq(fizzbuzz.fizzbuzz(n), "fizz");
    }
    
    function test_returns_buzz_when_divisible_by_five(uint256 n) public {
        vm.assume(n % 3 != 0);
        vm.assume(n % 5 == 0);
        assertEq(fizzbuzz.fizzbuzz(n), "buzz");
    }

    function test_returns_fizzbuzz_when_divisible_by_three_and_five(uint256 n) public {
        vm.assume(n % 3 == 0);
        vm.assume(n % 5 == 0);
        assertEq(fizzbuzz.fizzbuzz(n), "fizzbuzz");
    }

    function test_returns_number_as_string_otherwise(uint256 n) public {
        vm.assume(n % 3 != 0);
        vm.assume(n % 5 != 0);
        assertEq(fizzbuzz.fizzbuzz(n), Strings.toString(n));
    }
}
```

Note how we imported and used the `Strings.sol` library just like we did in our production code in order to convert integer values to strings in the last test case.

Forge will run 256 tests by default, but we can increase the number of fuzz runs with a configuration option in `foundry.toml`: 

```toml
[default]
src = 'src'
out = 'out'
libs = ['lib']
verbosity = 2
fuzz_runs = 5000
```

```bash
$ forge test
[⠊] Compiling...
No files changed, compilation skipped

Running 4 tests for src/test/FizzBuzz.t.sol:FizzBuzzTest
[PASS] test_returns_buzz_when_divisible_by_five(uint256) (runs: 5000, μ: 10755, ~: 10755)
[PASS] test_returns_fizz_when_divisible_by_three(uint256) (runs: 5000, μ: 10724, ~: 10724)
[PASS] test_returns_fizzbuzz_when_divisible_by_three_and_five(uint256) (runs: 5000, μ: 10635, ~: 10635)
[PASS] test_returns_number_as_string_otherwise(uint256) (runs: 5000, μ: 76811, ~: 98075)
Test result: ok. 4 passed; 0 failed; finished in 1.53s
```
