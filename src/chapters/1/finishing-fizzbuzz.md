# Finishing Fizzbuzz

Although our tests all pass, our code is still pretty naive. We've hardcoded `if` statements that will only work for the numbers `3` and `5`. Let's add a few more test cases to force ourselves to handle more than one value:

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

With the above tests in place, here's our failing test output:

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

Like many other languages, Solidity includes a `%` [modulo operator](https://docs.soliditylang.org/en/latest/types.html#modulo) that returns the remainder after dividing two numbers. We can use it to check for divisibility by testing whether the remainder is zero:

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
