# Writing the first test

Let's replace our intentionally failing test with our first real test. Values divisible by 3 should print "fizz":

```solidity
function test_returns_fizz_when_divisible_by_three() public {
    assertEq(fizzbuzz.fizzbuzz(3), "fizz");
}
```

Test driving our contract means we define our contract's interface when we write our tests. In this case, we'll expect to add a `fizzbuzz` function that takes an integer argument and returns a string. Let's give our tests another run:

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

The Solidity compiler fits nicely into a TDD workflow, since the compiler will usually point towards the next incremental step. In this case, we haven't  yet created a `fizzbuzz` function on our contract under test. Let's keep our workflow disciplined and add the simplest implementation that could possibly pass:

```solidity
contract FizzBuzz {
    function fizzbuzz(uint n) public returns (string memory) {
        return "fizz";
    }
}
```

And while we're here, let's take a brief detour to cover a few Solidity concepts...
