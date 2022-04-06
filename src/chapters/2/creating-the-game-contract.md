# Creating the game contract

Just like we did when we set up for Fizzbuzz, let's first create a new test file:

```bash
$ touch src/test/TicTacToken.t.sol
```
For now we'll just import ds-test and create an empty contract:

```solidity
// SPDX-License-Identifier: Apache-2.0
pragma solidity 0.8.10;

import "ds-test/test.sol";

contract TicTacTokenTest is DSTest {}
```

And a new contract for the game:

```bash
$ touch src/TicTacToken.sol
```

We'll start with an empty contract here, too:

```solidity
// SPDX-License-Identifier: Apache-2.0
pragma solidity 0.8.10;

contract TicTacToken {}
```
