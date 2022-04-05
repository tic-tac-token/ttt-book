# The final test

We have one more case to test, when the number is divisible by _both_ 3 and 5. Here's a test:

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

And finally, add the laziest solution possible, which completes our Fizzbuzz:

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
