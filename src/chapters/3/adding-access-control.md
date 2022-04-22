# Adding access control

It's great that we can interact with our Tic Tac Toe game contract, but hopefully you can already see the shortcomings of our current code: everyone else can interact withour contract, too! Anyone can mark a square on the board or clobber it altogether at any time by calling our contract's public functions. Rather than a public free-for-all, we'll want to restrict who's allowed to call the functions on our contract.

Fortunately, another property of the weird and wonderful Ethereum programming paradigm makes this possible. Since every state changing function call is a cryptographically signed transaction, we can tell _exactly_ who's calling a function! (Or at least, identify their public key/address). Solidity makes this easy by providing a special global variable called `msg.sender`.

> **Wallets, accounts, addresses, and keys**
>
> An _account_ is an entity with an ether balance that can send Ethereum transactions. There are two types of accounts: externally owned accounts and contracts. Every externally owned account (or "EOA") has a corresponding public and private _key_, and is controlled by the owner of its private key. Contract accounts do not have keys and are instead controlled by their deployed code.
>
> Every account has an _address_, a unique identifier made up of 42 hexadecimal characters, like `0xeD43C19583204FB9eFd041a4d9787bbE5c1965C3`. For an externally owned account, this address is derived from its public key. For a contract account, it's derived from the address of the contract creator and their transaction history. An address is not a public key, but it corresponds to one.
>
> A _wallet_ is a software application for managing an Ethereum account. It typically handles generating a keypair and corresponding address and securely storing the private key for one or many accounts.



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
$ forge test -m msg_sender

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

In the context of our tests, `msg.sender` has always been another contract (remember that our ds-test harness is itself a Solidity contract), but if an EOA were to call our `msgSender` function, it would return that EOA's address.
