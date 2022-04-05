# Symbolic testing

Property-based tests are like running many randomly generated unit test cases, but we can do even better using *symbolic execution*. Rather than picking a specific, concrete integer value of `n` for each test, symbolic execution uses a special VM that leaves the value `n` as an abstract representation and exhaustively explores every possible execution path. This is more like a proof than a test, since it covers every possible input. 

Foundry does not yet support symbolic execution and testing, although it's on the [roadmap](https://github.com/gakonst/foundry/issues/15). Instead we can use its predecessor [Dapptools](http://dapp.tools/) to write a few symbolic execution tests.

> **Installing Dapptools**
>
> If you want to install Dapptools and follow along, see the instructions [here](https://github.com/dapphub/dapptools#installation). Dapptools is a little less friendly than Foundry, and requires installing the [Nix](https://nixos.org/download.html) package manager.

To change our property-based tests to dapptools proofs, we can replace `test_` with `prove_` in the test name. Dapptools does not use the `vm.assume` cheatcode, but instead uses early returns inside each test function to filter out test cases that don't meet the test's assumptions. (Note that this means we've inverted the logic for each conditional.)

```bash
contract FizzBuzzTest is DSTest {
    FizzBuzz internal fizzbuzz;

    function setUp() public {
        fizzbuzz = new FizzBuzz();
    }

    function prove_returns_fizz_when_divisible_by_three(uint256 n) public {
        if(n % 3 != 0) return;
        if(n % 5 == 0) return;
        assertEq(fizzbuzz.fizzbuzz(n), "fizz");
    }
    
    function prove_returns_buzz_when_divisible_by_five(uint256 n) public {
        if(n % 3 == 0) return;
        if(n % 5 != 0) return;
        assertEq(fizzbuzz.fizzbuzz(n), "buzz");
    }

    function prove_returns_fizzbuzz_when_divisible_by_three_and_five(uint256 n) public {
        if(n % 3 != 0) return;
        if(n % 5 != 0) return;
        assertEq(fizzbuzz.fizzbuzz(n), "fizzbuzz");
    }
}
```

Here's what our test results look like using Dapptools:

```bash
$ DAPP_SOLC_VERSION=0.8.10 DAPP_REMAPPINGS=$(cat remappings.txt) dapp test
Temporarily installing solc-0.8.10...
Running 3 tests for src/test/FizzBuzz.t.sol:FizzBuzzTest
[PASS] prove_returns_fizzbuzz_when_divisible_by_three_and_five(uint256)
[PASS] prove_returns_fizz_when_divisible_by_three(uint256)
[PASS] prove_returns_buzz_when_divisible_by_five(uint256)
```

You'll notice these are significantly slower to run than unit or property-based tests. On my machine, these take several minutes to run. But they give a much stronger guarantee about the correctness of our code. (They are slower because they are doing a lot more computation using an [SMT solver](https://en.wikipedia.org/wiki/Satisfiability_modulo_theories) under the hood).

You may also notice that we've omitted a test case: proving the case for numbers that are not multiples of 3 or 5 is too complex. Symbolic execution is great for a narrow set of provable properties without too much branching logic, like simple arithmetic functions, but for complex functions, proofs can become infeasible with just a little additional complexity. In this case, the call to `Strings.toString`, which includes several internal branching statements, adds too much complexity for our proof to complete. 

However, it's often possible to rewrite or simplify functions into something more tractable. Imagine if our `fizzbuzz` function returned a number instead of a string. This is logically equivalent, except we're omitting the string conversion step in the final case:

```solidity
contract FizzBuzz {

    function fizzbuzz(uint256 n) public pure returns (uint256) {
        if (n % 3 == 0 && n % 5 == 0) {
            return 1;
        }
        else if (n % 3 == 0) {
            return 2;
        }
        else if (n % 5 == 0) {
            return 3;
        } else {
            return 4;
        }
    }
}
```

Now, we can prove the following test, in about the same time as it takes to run the others:

```solidity
    function prove_returns_fizzbuzz_otherwise(uint256 n) public {
        if(n % 3 == 0) return;
        if(n % 5 == 0) return;
        assertEq(fizzbuzz.fizzbuzz(n), 4);
    }
```

This ability to "upgrade" our tests from one specific input case to many randomized cases to a generalized proof is a very cool and powerful feature of Foundry and Dapptools.
