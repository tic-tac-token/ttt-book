# Reading the contract under test
Similarly, Let's take a look at `Greeter.sol`, the contract under test, and see what we can deduce. Here it is in full:

```solidity
// SPDX-License-Identifier: Apache-2.0
pragma solidity 0.8.10;

import "openzeppelin-contracts/contracts/access/Ownable.sol";

contract Greeter is Ownable {
    string public greeting;

    constructor(string memory _greeting) {
        greeting = _greeting;
    }

    function greet() public view returns (string memory) {
        return _buildGreeting("world");
    }

    function greet(string memory name) public view returns (string memory) {
        return _buildGreeting(name);
    }

    function setGreeting(string memory _greeting) public onlyOwner {
        greeting = _greeting;
    }

    function _buildGreeting(string memory name) internal view returns (string memory) {
        return string(abi.encodePacked(greeting, ", ", name, "!"));
    }
}
```

We can import code from external files:

```solidity
import "openzeppelin-contracts/contracts/access/Ownable.sol";
```

Use inheritance:

```solidity
contract Greeter is Ownable {
}
```

Define variables with visibility:

```solidity
    string public greeting;
```

Create functions with arguments:

```solidity
    function greet(string memory name) public view returns (string memory) {
        return _buildGreeting(name);
    }
```

...and without them:

```solidity
    function greet() public view returns (string memory) {
        return _buildGreeting("world");
    }
```

Set variables:

```solidity
    function setGreeting(string memory _greeting) public onlyOwner {
        greeting = _greeting;
    }
```

...and call internal functions:

```solidity
    function greet(string memory name) public view returns (string memory) {
        return _buildGreeting(name);
    }

    function _buildGreeting(string memory name) internal view returns (string memory) {
        return string(abi.encodePacked(greeting, ", ", name, "!"));
    }
```

Some of this, like the keyword `contract`, types like `(string memory)`, and the `abi.encodePacked` function, may look a little esoteric, but most of this code should be pretty legible. Again, if you squint hard enough, it kind of looks like Javascript.
