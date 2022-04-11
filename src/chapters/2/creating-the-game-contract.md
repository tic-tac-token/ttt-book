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

We'll do the same for the game contract. Create a new file:

```bash
$ touch src/TicTacToken.sol
```

And we can start with an empty contract here, too:

```solidity
// SPDX-License-Identifier: Apache-2.0
pragma solidity 0.8.10;

contract TicTacToken {}
```
