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

Let's replace this intentional failure with our first real test:

```solidity
function test_returns_fizz_when_divisible_by_three() public {
    assertE1(fizzbuzz.fizzbuzz(3), "fizz");
}
```

Test driving our contract means we define the interface we want as we write our tests. In this case, we'll add a `fizzbuzz` function that takes an integer argument and returns a string. Let's give our tests another run:

```bash
$ forge test
[⠊] Compiling...
[⠢] Compiling 1 files with 0.8.10
[⠆] Solc finished in 9.24ms
Error: 
   0: Compiler run failed
      TypeError: Member "fizzbuzz" not found or not visible after argument-dependent lookup in contract FizzBuzz.
        --> /Users/ecm/Projects/forge-template/src/test/FizzBuzz.t.sol:15:18:
         |
      15 |         assertEq(fizzbuzz.fizzbuzz(3), "fizz");
         |                  ^^^^^^^^^^^^^^^^^
   0: 

Location:
   cli/src/cmd/utils.rs:43
```

The Solidity compiler fits nicely into a TDD workflow, since the compiler will usually point towards the next incremental step. In this case, we haven't created a `fizzbuzz` funciton on our contract under test. Let's keep it disciplined and add the simplest implementation that could possibly pass:

```solidity
contract FizzBuzz {
    function fizzbuzz(uint n) public returns (string memory) {
        return "fizz";
    }
}
```

Let's stop and cover a bit of Solidity syntax while we're here: [visibility](https://docs.soliditylang.org/en/latest/contracts.html#visibility-and-getters), [value types](https://docs.soliditylang.org/en/latest/types.html#value-types), and [reference types](https://docs.soliditylang.org/en/latest/types.html#reference-types).

#### Visibility
We've defined `fizzbuzz` as a `public` function, which means it can be called both internally by other methods in our contract and externally through message sends. There are a few other function visibility modifiers: `external` functions can be called by other contracts but not internally, `internal` functions can only be accessed internally, and `private` functions can only be accessed internally *and* are not visible to derived contracts. Variables have `public`, `internal`, and `private` visibility, too.

#### Value types
Our `fizzbuzz` function takes one parameter `n` as an argument and returns a string. The input parameter `n` is a `uint`, or unsigned 256-bit integer. "Unsigned" means this integer type represents a non-negative integer value. Integers in Solidity are _value types_, i.e. always copied and passed by value when they are used in arguments and assignments.

256-bit integers are very large: a `uint256` can store a value as large as \\( 2^{256}-1 \\), which is way bigger than the 64-bit integers used by default in most other common programming languages. 

To compare the maximum value of the two types, let's jump into a Python shell and look at the difference:

```bash
$ python
>>> (2 ** 64) - 1
18446744073709551615
>>> (2 ** 256) - 1
115792089237316195423570985008687907853269984665640564039457584007913129639935
```

The `uint` type is implicitly interpreted as `uint256`, but it's considered good Solidity style to always be explicit and prefer using `uint256` to bare `uint`.

#### Reference types
The return value of our function is `(string memory)`, a _reference type_. Structs, arrays, and mappings are all reference types in Solidity. (Strings are secretly arrays of bytes under the hood, so they are reference types too). Unlike value types, which are copied each time they are used, reference types are passed by reference, so we have to be more careful about how they are used and modified to avoid unexpected mutations.

When we declare a reference type, we must also always declare the "data area" where it will be stored. There are three options: `calldata`, `memory`, and `storage`. In the case of our return value, we're using `memory`. 

`calldata` is a special, immutable, super-temporary location for function arguments. When you can get away with using it, `calldata` is a great location because it's immutable, avoids copies, and is cheap to use.

`memory` is a temporary location analogous to runtime memory. Every function call gets access to a freshly cleared chunk of memory that can expand as necessary. Writing to memory is much cheaper than writing to `storage`, but it still costs gas to read and write.

`storage` is a permanent location that is persistent between function calls. It is expensive to read and very expensive to initialize and write. (This is for good reason: any data we write to storage will be replicated on every node in the network and stored forever!)

Onward. Let's run our tests again now that we understand our own code.

```bash
$ forge test
[⠊] Compiling...
[⠢] Compiling 2 files with 0.8.10
[⠆] Solc finished in 75.22ms
Compiler run successful (with warnings)
Warning: Unused function parameter. Remove or comment out the variable name to silence this warning.
 --> /Users/ecm/Projects/forge-template/src/FizzBuzz.sol:6:23:
  |
6 |     function fizzbuzz(uint n) public returns (string memory) {
  |                       ^^^^^^

Warning: Function state mutability can be restricted to pure
 --> /Users/ecm/Projects/forge-template/src/FizzBuzz.sol:6:5:
  |
6 |     function fizzbuzz(uint n) public returns (string memory) {
  |     ^ (Relevant source part starts here and spans across multiple lines).

Running 1 test for src/test/FizzBuzz.t.sol:FizzBuzzTest
[PASS] test_returns_fizz_when_divisible_by_three() (gas: 6984)
Test result: ok. 1 passed; 0 failed; finished in 249.13µs
```

Success! Our first passing test.

The Solidity compiler has also passed on some helpful warnings. First, we're not yet using the parameter we've passed to the `fizzbuzz` function, so we can omit giving it a name. (This looks weird, but fine). Second, since our function doesn't read or mutate any state, we can add an additional function modifier and declare it `pure`. It's a good habit to fix compiler warnings as we go. Here's what our code looks like after following these recommendations:

```solidity
contract Fizzbuzz {
    function fizzbuzz(uint256) public pure returns (string memory) {
        return "fizz";
    }
}
```

Let's follow the usual TDD workflow and add a new test to drive us forward:

```solidity
function test_returns_buzz_when_divisible_by_five() public {
    assertEq(fizzbuzz.fizzbuzz(5), "buzz");
}
```

Run it and see it fail:

```bash
$ forge test
[⠊] Compiling...
No files changed, compilation skipped

Running 2 tests for src/test/FizzBuzz.t.sol:FizzBuzzTest
[FAIL] test_returns_buzz_when_divisible_by_five() (gas: 17351)
Logs:
  Error: a == b not satisfied [string]
    Value a: fizz
    Value b: buzz

[PASS] test_returns_fizz_when_divisible_by_three() (gas: 6984)
Test result: FAILED. 1 passed; 1 failed; finished in 1.35ms

Failed tests:
[FAIL] test_returns_buzz_when_divisible_by_five() (gas: 17351)

Encountered a total of 1 failing tests, 1 tests succeeded
```

...and update our production code. Again, let's write the simplest thing that could possibly work:

```solidity
contract Fizzbuzz {
    function fizzbuzz(uint256 n) public pure returns (string memory) {
        if (n == 5) {
            return "buzz";
        }
        return "fizz";
    }
}
```

And watch the tests pass...

```bash
$ forge test
[⠊] Compiling...
[⠢] Compiling 2 files with 0.8.10
[⠆] Solc finished in 77.56ms
Compiler run successful

Running 2 tests for src/test/FizzBuzz.t.sol:FizzBuzzTest
[PASS] test_returns_buzz_when_divisible_by_five() (gas: 7034)
[PASS] test_returns_fizz_when_divisible_by_three() (gas: 7024)
Test result: ok. 2 passed; 0 failed; finished in 319.33µs
```

One more test:

```solidity
function test_returns_number_as_string_otherwise() public {
    assertEq(fizzbuzz.fizzbuzz(7), "7");
}
```

And failure:

```bash
$ forge test
[⠊] Compiling...
[⠢] Compiling 1 files with 0.8.10
[⠆] Solc finished in 84.71ms
Compiler run successful

Running 3 tests for src/test/FizzBuzz.t.sol:FizzBuzzTest
[PASS] test_returns_buzz_when_divisible_by_five() (gas: 7056)
[PASS] test_returns_fizz_when_divisible_by_three() (gas: 7046)
[FAIL] test_returns_number_as_string_otherwise() (gas: 17380)
Logs:
  Error: a == b not satisfied [string]
    Value a: fizz
    Value b: 7

Test result: FAILED. 2 passed; 1 failed; finished in 1.58ms

Failed tests:
[FAIL] test_returns_number_as_string_otherwise() (gas: 17380)
```

We're in uncharted territory now: we need to convert our `uint256` input into a `string memory`. Maybe we can just try casting it to another type with the `string` function?

```solidity
contract FizzBuzz {

    function fizzbuzz(uint256 n) public pure returns (string memory) {
        if (n == 5) {
            return "buzz";
        }
        if (n == 3) {
            return "fizz";
        }
        return string(n);
    }
}
```

Let's give it a try:

```bash
$ forge test
[⠊] Compiling...
[⠒] Compiling 2 files with 0.8.10
[⠢] Solc finished in 8.57ms
Error: 
   0: Compiler run failed
      TypeError: Explicit type conversion not allowed from "uint256" to "string memory".
        --> /Users/ecm/Projects/forge-template/src/FizzBuzz.sol:13:16:
         |
      13 |         return string(n);
         |                ^^^^^^^^^
   0: 

Location:
   cli/src/cmd/utils.rs:43
```

Nope. The good news is that the Solidity compiler is pretty helpful in cases like these.

There's no built in integer-to-string conversion in Solidity, so we have a few options: write it ourselves, copy-paste [some other implementation](https://stackoverflow.com/questions/47129173/how-to-convert-uint-to-string-in-solidity), or import a library. 

Let's use a library. [OpenZeppelin Contracts](https://openzeppelin.com/contracts/) includes a string utilities library that does exactly what we need. Here's how to import and use it:

```solidity
import "openzeppelin-contracts/contracts/utils/Strings.sol";

contract FizzBuzz {

    function fizzbuzz(uint256 n) public pure returns (string memory) {
        if (n == 5) {
            return "buzz";
        }
        if (n == 3) {
            return "fizz";
        }
        return Strings.toString(n);
    }
}
```

You can read more about libraries in the [Solidity docs](https://docs.soliditylang.org/en/latest/contracts.html#libraries).

Let's try our tests again:

```bash
$ forge test
[⠊] Compiling...
[⠢] Compiling 3 files with 0.8.10
[⠆] Solc finished in 107.12ms
Compiler run successful

Running 3 tests for src/test/FizzBuzz.t.sol:FizzBuzzTest
[PASS] test_returns_buzz_when_divisible_by_five() (gas: 7056)
[PASS] test_returns_fizz_when_divisible_by_three() (gas: 7071)
[PASS] test_returns_number_as_string_otherwise() (gas: 7820)
Test result: ok. 3 passed; 0 failed; finished in 1.57ms
```

Success! Of course, although our tests all pass, our code is still pretty naive. Let's add a few more test cases to force ourselves to handle more than one value:

```solidity
    function test_returns_fizz_when_divisible_by_three() public {
        assertEq(fizzbuzz.fizzbuzz(3), "fizz");
        assertEq(fizzbuzz.fizzbuzz(6), "fizz");
        assertEq(fizzbuzz.fizzbuzz(27), "fizz");
    }

    function test_returns_buzz_when_divisible_by_five() public {
        assertEq(fizzbuzz.fizzbuzz(5), "buzz");
        assertEq(fizzbuzz.fizzbuzz(10), "buzz");
        assertEq(fizzbuzz.fizzbuzz(175), "buzz");
    }
```

Here's our failing test output:

```bash
$ forge test
[⠊] Compiling...
[⠢] Compiling 2 files with 0.8.10
[⠆] Solc finished in 105.78ms
Compiler run successful

Running 3 tests for src/test/FizzBuzz.t.sol:FizzBuzzTest
[FAIL] test_returns_buzz_when_divisible_by_five() (gas: 33029)
Logs:
  Error: a == b not satisfied [string]
    Value a: 10
    Value b: buzz
  Error: a == b not satisfied [string]
    Value a: 175
    Value b: buzz

[FAIL] test_returns_fizz_when_divisible_by_three() (gas: 31897)
Logs:
  Error: a == b not satisfied [string]
    Value a: 6
    Value b: fizz
  Error: a == b not satisfied [string]
    Value a: 27
    Value b: fizz

[PASS] test_returns_number_as_string_otherwise() (gas: 7820)
Test result: FAILED. 1 passed; 2 failed; finished in 1.40ms

Failed tests:
[FAIL] test_returns_buzz_when_divisible_by_five() (gas: 33029)
[FAIL] test_returns_fizz_when_divisible_by_three() (gas: 31897)

Encountered a total of 2 failing tests, 1 tests succeeded
```

Like many other languages, Solidity includes a `%` [modulo operator](https://docs.soliditylang.org/en/latest/types.html#modulo) that returns the remainder after dividing two numbers. We can use it to check for divisibility:

```solidity
contract FizzBuzz {

    function fizzbuzz(uint256 n) public pure returns (string memory) {
        if (n % 5 == 0) {
            return "buzz";
        }
        if (n % 3 == 0) {
            return "fizz";
        }
        return Strings.toString(n);
    }
}
```

Let's try it out:

```bash
$ forge test
[⠊] Compiling...
[⠢] Compiling 2 files with 0.8.10
[⠆] Solc finished in 105.16ms
Compiler run successful

Running 3 tests for src/test/FizzBuzz.t.sol:FizzBuzzTest
[PASS] test_returns_buzz_when_divisible_by_five() (gas: 11978)
[PASS] test_returns_fizz_when_divisible_by_three() (gas: 12178)
[PASS] test_returns_number_as_string_otherwise() (gas: 7916)
Test result: ok. 3 passed; 0 failed; finished in 1.31ms
```

Success! Tests pass.

OK, one more case to test:

```solidity
function test_returns_fizzbuzz_when_divisible_by_three_and_five() public {
    assertEq(fizzbuzz.fizzbuzz(15), "fizzbuzz");
}
```

Let's see the expected failure:

```bash
$ forge test
[⠊] Compiling...
[⠢] Compiling 1 files with 0.8.10
[⠆] Solc finished in 111.56ms
Compiler run successful

Running 4 tests for src/test/FizzBuzz.t.sol:FizzBuzzTest
[PASS] test_returns_buzz_when_divisible_by_five() (gas: 11956)
[PASS] test_returns_fizz_when_divisible_by_three() (gas: 12156)
[FAIL] test_returns_fizzbuzz_when_divisible_by_three_and_five() (gas: 37420)
Logs:
  Error: a == b not satisfied [string]
    Value a: buzz
    Value b: fizzbuzz
  Error: a == b not satisfied [string]
    Value a: buzz
    Value b: fizzbuzz
  Error: a == b not satisfied [string]
    Value a: buzz
    Value b: fizzbuzz

[PASS] test_returns_number_as_string_otherwise() (gas: 7939)
Test result: FAILED. 3 passed; 1 failed; finished in 1.36ms

Failed tests:
[FAIL] test_returns_fizzbuzz_when_divisible_by_three_and_five() (gas: 37420)

Encountered a total of 1 failing tests, 3 tests succeeded
```

Add the laziest solution possible:

```solidity
// SPDX-License-Identifier: Apache-2.0
pragma solidity 0.8.10;

import "openzeppelin-contracts/contracts/utils/Strings.sol";
contract FizzBuzz {

    function fizzbuzz(uint256 n) public pure returns (string memory) {
        if (n % 3 == 0 && n % 5 == 0) {
            return "fizzbuzz";
        }
        if (n % 3 == 0) {
            return "fizz";
        }
        if (n % 5 == 0) {
            return "buzz";
        }
        return Strings.toString(n);
    }
}
```

```bash
$ forge test
[⠊] Compiling...
[⠢] Compiling 2 files with 0.8.10
[⠆] Solc finished in 116.50ms
Compiler run successful

Running 4 tests for src/test/FizzBuzz.t.sol:FizzBuzzTest
[PASS] test_returns_buzz_when_divisible_by_five() (gas: 12478)
[PASS] test_returns_fizz_when_divisible_by_three() (gas: 12429)
[PASS] test_returns_fizzbuzz_when_divisible_by_three_and_five() (gas: 12255)
[PASS] test_returns_number_as_string_otherwise() (gas: 8039)
Test result: ok. 4 passed; 0 failed; finished in 1.13ms
```
