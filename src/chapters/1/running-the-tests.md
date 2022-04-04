# Running the tests
Run `forge test` to compile the project and run the tests:

```bash
$ forge test
[⠊] Compiling...
[⠆] Compiling 3 files with 0.8.10
[⠔] Solc finished in 225.61ms
Compiler run successful

Running 5 tests for src/test/Greeter.t.sol:GreeterTest
[PASS] test_custom_greeting() (gas: 10206)
[PASS] test_default_greeting() (gas: 9822)
[PASS] test_get_greeting() (gas: 9813)
[PASS] test_non_owner_cannot_set_greeting() (gas: 12202)
[PASS] test_set_greeting() (gas: 18728)
Test result: ok. 5 passed; 0 failed; finished in 592.04µs
```

Looks good—everything passed. Let's explore the test file a bit further.
