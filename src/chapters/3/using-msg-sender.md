# Using `msg.sender`

Inside a smart contract, Solidity provides a special, [globally accessible](https://docs.soliditylang.org/en/latest/units-and-global-variables.html#block-and-transaction-properties) `msg.sender` variable, which stores the address of the current caller. If the caller is an EOA, `msg.sender` will be the caller account's address. If the caller is another contract, `msg.sender` will be that contract's address.

Let's write a test to explore how it works:

```solidity
    function test_msg_sender() public {
        // Not sure what this will return yet...
        // Let's try the zero address and see.
        assertEq(ttt.msgSender(), address(0));
    }
```

Run it to find out:

```solidity
$ forge test -m msg_sender -vv

Running 1 test for src/test/TicTacToken.t.sol:TicTacTokenTest
[FAIL] test_msg_sender() (gas: 16368)
Logs:
  Error: a == b not satisfied [address]
    Expected: 0x0000000000000000000000000000000000000000
      Actual: 0xb4c79dab8f259c7aee6e5b2aa729821864227e84

Test result: FAILED. 0 passed; 1 failed; finished in 2.35ms

Failed tests:
[FAIL] test_msg_sender() (gas: 16368)
```

In this context, `msg.sender` is the address `0xb4c79dab8f259c7aee6e5b2aa729821864227e84`, which corresponds to the address of the `TicTacTokenTest` contract. Since our test harness contract is calling the function, it's the `msg.sender`.

We can access the address of the current contract by calling `address(this)`. Let's update our test:

```solidity
    function test_msg_sender() public {
        assertEq(ttt.msgSender(), address(this));
    }
```

Success!

```solidity
Running 1 test for src/test/TicTacToken.t.sol:TicTacTokenTest
[PASS] test_msg_sender() (gas: 5502)
Test result: ok. 1 passed; 0 failed; finished in 1.64ms
```

Now let's add another layer to explore how `msg.sender` changes based on the current caller. At the top of our test file, let's create _another_ contract, `Caller`. We'll give it access to our game contract and add a function `call` that calls our game's `msgSender()` function and returns the address:

```solidity
contract Caller {

    TicTacToken internal ttt;

    constructor(TicTacToken _ttt) {
        ttt = _ttt;
    }
    function call() public returns (address) {
        return ttt.msgSender();
    }
}
```

Now let's create two instances of the `Caller` contract in our test. Any guesses on the value of `msg.sender` here?

```solidity
    function test_msg_sender() public {
        Caller caller1 = new Caller(ttt);
        Caller caller2 = new Caller(ttt);

        assertEq(ttt.msgSender(), address(this));

        assertEq(caller1.call(), address(0));
        assertEq(caller2.call(), address(0));
    }
```

Let's run the test to find out:

```bash
$ forge test -m msg_sender
Running 1 test for src/test/TicTacToken.t.sol:TicTacTokenTest
[FAIL] test_msg_sender() (gas: 256093)
Logs:
  Error: a == b not satisfied [address]
    Expected: 0x0000000000000000000000000000000000000000
      Actual: 0x185a4dc360ce69bdccee33b3784b0282f7961aea
  Error: a == b not satisfied [address]
    Expected: 0x0000000000000000000000000000000000000000
      Actual: 0xefc56627233b02ea95bae7e19f648d7dcd5bb132

Test result: FAILED. 0 passed; 1 failed; finished in 826.17µs

Failed tests:
[FAIL] test_msg_sender() (gas: 256093)

Encountered a total of 1 failing tests, 0 tests succeeded
```

Notice that the values are different for `caller1` and `caller2`. Since the caller is now another, intermediate contract, `msg.sender` is the address of `caller1` and `caller2` respectively. We can access them in the tests like so:

```solidity
    function test_msg_sender() public {
        Caller caller1 = new Caller(ttt);
        Caller caller2 = new Caller(ttt);

        assertEq(ttt.msgSender(), address(this));

        assertEq(caller1.call(), address(caller1));
        assertEq(caller2.call(), address(caller2));
    }
```

Let's run the tests and make sure:

```bash
$ forge test -m msg_sender
Running 1 test for src/test/TicTacToken.t.sol:TicTacTokenTest
[PASS] test_msg_sender() (gas: 239187)
Test result: ok. 1 passed; 0 failed; finished in 829.67µs
```

In the context of our tests, `msg.sender` has always been another contract (remember that our ds-test test harness is itself a Solidity contract), but if an EOA were to call our `msgSender` function, it would return that EOA's address.
