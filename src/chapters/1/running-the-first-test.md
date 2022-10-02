# Running the first test

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

Success! Our first passing test. The Solidity compiler has also passed on some helpful warnings:

```
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
  ```

First, we're not yet using the parameter we've passed to the `fizzbuzz` function, so we can omit giving it a name. (This looks weird, but fine). 

Second, since our function doesn't read or mutate any state, we can add an additional function modifier and declare it `pure`. It's good to build the habit of fixing any compiler warnings as we go. We'll also use the more explicit `uint256` instead of `uint`. 

Here's what our full contract code looks like after following these recommendations:

```solidity
// SPDX-License-Identifier: Apache-2.0
pragma solidity 0.8.10;

contract FizzBuzz {
    function fizzbuzz(uint256) public pure returns (string memory) {
        return "fizz";
    }
}
```
