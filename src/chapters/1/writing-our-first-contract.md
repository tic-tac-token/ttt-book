# Writing our first contract
Let's take our first steps into writing Solidity by [test driving](http://wiki.c2.com/?TestDrivenDevelopment) a Fizzbuzz contract. Our Fizzbuzz function should take an integer argument and:

- Return "fizz" for numbers divisible by 3
- Return "buzz" for numbers divisible by 5
- Return "fizzbuzz" for numbers divisible by both 3 and 5
- Return the input number as a string for all other cases.

Let's start by setting up a new test contract. Create `FizzBuzz.t.sol` alongside `Greeter.t.sol`:

```bash
$ touch src/test/FizzBuzz.t.sol
```

We'll start with a test that intentionally fails in order to make sure everything's connected correctly:

```solidity
// SPDX-License-Identifier: Apache-2.0
pragma solidity 0.8.10;

import "ds-test/test.sol";
import "../FizzBuzz.sol";

contract FizzBuzzTest is DSTest {
    FizzBuzz internal fizzbuzz;

    function setUp() public {
        fizzbuzz = new FizzBuzz();
    }

    function test_math_is_broken() public {
        uint256 two = 1 + 1;
        assertEq(two, 3);
    }
}
```

One short note on style: it's conventional to use `mixedCase` for Solidity function and variable names, but I like to intentionally break this rule for test functions and use `snake_case` instead. This helps distinguish them from production code when reading test output and function traces.

Let's give our newly created tests a spin:

```bash
$ forge test                   
[⠊] Compiling...
[⠒] Unable to resolve import: "../FizzBuzz.sol" with remappings:
    ds-test/=/Users/ecm/Projects/forge-template/lib/ds-test/src/
    forge-std/=/Users/ecm/Projects/forge-template/lib/forge-std/src/
    openzeppelin-contracts/=/Users/ecm/Projects/forge-template/lib/openzeppelin-contracts/
    src/=/Users/ecm/Projects/forge-template/src/
[⠆] Compiling 2 files with 0.8.10
[⠰] Solc finished in 9.06ms
Error: 
   0: Compiler run failed
      ParserError: Source "/Users/ecm/Projects/forge-template/src/FizzBuzz.sol" not found: File not found.
       --> /Users/ecm/Projects/forge-template/src/test/FizzBuzz.t.sol:5:1:
        |
      5 | import "../FizzBuzz.sol";
        | ^^^^^^^^^^^^^^^^^^^^^^^^^
   0: 

Location:
   cli/src/cmd/utils.rs:43
```

Oops—we forgot to create our contract under test! Fortunately, the Solidity compiler outbut is usually very helpful—here it points to the location of the missing import. Let's create `FizzBuzz.sol` and try again:

```bash
$ touch src/FizzBuzz.sol
```

For now, we'll just create an empty contract:

```solidity
// SPDX-License-Identifier: Apache-2.0
pragma solidity 0.8.10;

contract FizzBuzz {
}
```

With our empty production contract in place, let's try again:

```bash
forge test
[⠊] Compiling...
[⠢] Compiling 2 files with 0.8.10
[⠆] Solc finished in 32.87ms
Compiler run successful

Running 1 test for src/test/FizzBuzz.t.sol:FizzBuzzTest
[FAIL] test_math_is_broken() (gas: 10940)
Logs:
  Error: a == b not satisfied [uint]
    Expected: 3
      Actual: 2

Test result: FAILED. 0 passed; 1 failed; finished in 1.49ms

Failed tests:
[FAIL] test_math_is_broken() (gas: 10940)

Encountered a total of 1 failing tests, 0 tests succeeded
```

Success! Math is not actually broken, and our test failed as it should. Forge printed the expected and actual values to help us diagnose the failure. 
