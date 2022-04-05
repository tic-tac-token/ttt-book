# Test driving Fizzbuzz

Let's follow the usual TDD workflow and add a new test to drive us forward. Passing the number 5 should print "buzz":

```solidity
function test_returns_buzz_when_divisible_by_five() public {
    assertEq(fizzbuzz.fizzbuzz(5), "buzz");
}
```

Run the test and see it fail:

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

...and finally, update our production code. Again, let's write the simplest thing that could possibly work:

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

Looks good. Let's add one more test. Numbers that aren't divisible by either 3 or 5 should return themselves as a string:

```solidity
function test_returns_number_as_string_otherwise() public {
    assertEq(fizzbuzz.fizzbuzz(7), "7");
}
```

Here's the failure:

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

So how do we convert this value to a string?
