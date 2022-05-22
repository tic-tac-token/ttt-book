# Creating an NFT

We've read the ERC721 spec and explored a real world contract. Now let's add an NFT as a component of our game. We haven't created a frontend for our Tic Tac Toe game yet, but we can use an NFT as a primitive UI. We can issue a token to each player at the start of every new game, and use dynamic metadata to display the state of the game board as an image. Players will be able to view their tokens in a wallet or on an NFT marketplace to see the current state of their ongoing games.

As always, let's start with the tests. We'll create a new `test/NFT.t.sol`, with a couple tests for our new token's name and symbol. Since the interface is similar, this should look pretty close to the tests for our ERC20:

```solidity
// SPDX-License-Identifier: Apache-2.0
pragma solidity 0.8.10;

import "ds-test/test.sol";
import "forge-std/Vm.sol";

import "../NFT.sol";

contract TicTacTokenTest is DSTest {
    Vm internal vm = Vm(HEVM_ADDRESS);

    NFT internal nft;

    function setUp() public {
        nft = new NFT();
    }

    function test_token_name() public {
        assertEq(nft.name(), "Tic Tac Token NFT");
    }

    function test_token_symbol() public {
        assertEq(nft.symbol(), "TTT NFT");
    }

}
```

We'll also create an empty `NFT.sol`:

```solidity
// SPDX-License-Identifier: Apache-2.0
pragma solidity 0.8.10;

contract NFT {}
```

Just like our ERC20, we'll need to import and inherit from the OpenZeppelin ERC721 base contract that provides these metadata functions:

```bash
$ forge test
Error:
   0: Compiler run failed
      TypeError: Member "name" not found or not visible after argument-dependent lookup in contract NFT.
        --> /Users/ecm/Projects/ttt-book-code/src/test/NFT.t.sol:19:18:
         |
      19 |         assertEq(nft.name(), "Tic Tac Token NFT");
         |                  ^^^^^^^^
```

Let's import the [ERC721 base contract](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721) and pass through the name and symbol constructor args:

```
import "openzeppelin-contracts/contracts/token/ERC721/ERC721.sol";

contract NFT is ERC721 {

    constructor() ERC721("Tic Tac Token NFT", "TTT NFT") {}

}
```

Now we're all set to start building our new token:

```bash
$ forge test --match-path src/test/NFT.t.sol
Compiler run successful

Running 2 tests for src/test/NFT.t.sol:TicTacTokenTest
[PASS] test_token_name() (gas: 9745)
[PASS] test_token_symbol() (gas: 9753)
Test result: ok. 2 passed; 0 failed; finished in 1.67ms
```

> **Filtering tests**
>
> Our test suite is getting pretty big, and we may not want to run all the tests all the time. Forge provides a few command line flags to filter test runs: `--match-test` will run only test functions matching a specified regex pattern, `--match-contract` will run only tests in contracts that match a given regex, and `--match-path` will only run tests from files that match a given glob pattern.
>

