# Importing a library

We're in uncharted territory now: we need to convert our `uint256` input into a `string memory`. Maybe we can just try converting it to another type with the `string` function?

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

Nope. The good news is that the Solidity compiler is pretty helpful in cases like these. It's told us that type conversion from `uint256` to string is not allowed. 

There's no built in integer-to-string conversion in Solidity, so we have a few options: write it ourselves, copy-paste [some other implementation](https://stackoverflow.com/questions/47129173/how-to-convert-uint-to-string-in-solidity), or import a library. 

Let's use a library. [OpenZeppelin Contracts](https://openzeppelin.com/contracts/) includes a string utilities library that does exactly what we need. 

We'll first need to install it using `forge`. Running `forge install` will install a repository from Github as a git submodule in our project's `lib/` directory:

```bash
$ forge install OpenZeppelin/openzeppelin-contracts
```

Once it's installed, here's how to import and use it:

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

Success! 
