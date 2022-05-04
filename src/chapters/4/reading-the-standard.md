# Reading the standard

The ERC20 standard is a short, simple read. [Take a look for yourself](https://eips.ethereum.org/EIPS/eip-20). Here are the Solidity function signatures of the whole interface:

```solidity
function name() public view returns (string)
function symbol() public view returns (string)
function decimals() public view returns (uint8)
function totalSupply() public view returns (uint256)

function balanceOf(address _owner) public view returns (uint256 balance)
function transfer(address _to, uint256 _value) public returns (bool success)

function transferFrom(address _from, address _to, uint256 _value) public returns (bool success)
function approve(address _spender, uint256 _value) public returns (bool success)
function allowance(address _owner, address _spender) public view returns (uint256 remaining)
```

An ERC20 token is a smart contract that conforms to this interface. Let's walk through each of the functions.

First up are a few functions that return metadata about the token:

```solidity
function name() public view returns (string)
function symbol() public view returns (string)
function decimals() public view returns (uint8)
function totalSupply() public view returns (uint256)
```

- `name` returns a human-readable name, like "Dai Stablecoin."
- `symbol` returns a short symbol, like "DAI"
- `decimals` returns the number of decimal places used to denominate the token. Most ERC20 tokens use 18 decimals, the same number used to subdivide Ether.
- `totalSupply` returns the total supply of the token.

Next, we have two functions related to reading balances and sending tokens:

```solidity
function balanceOf(address _owner) public view returns (uint256 balance)
```

The `balanceOf` function returns the token balance of the account with address `_owner`.

```solidity
function transfer(address _to, uint256 _value) public returns (bool success)
```

The `transfer` function sends some quantity of tokens `_value` from the caller account's address to the account with address `_to`. It's up to the implementation to keep track of balances and transfers internally in whatever way makes sense.

Finally, we have three functions that enable third parties to send tokens on behalf of a user. These are important in order to allow smart contracts to withdraw and spend tokens on behalf of externally owned accounts. Most of the essential complexity in ERC20 lives here:

```solidity
function transferFrom(address _from, address _to, uint256 _value) public returns (bool success)
```

`transferFrom` is a third-party transfer sending `_value` tokens from the `_from` address to the `_to` address. Before a third party can call `transferFrom`, the `_from` user must first call the next function:

```solidity
function approve(address _spender, uint256 _value) public returns (bool success)
```

`approve` allows the `_spender` address to withdraw and transfer a quantity `_value` of tokens. The `_spender` may withdraw multiple times, up to the `_value` amount. This approved amount is called an "allowance," which leads to the next function:

```solidity
function allowance(address _owner, address _spender) public view returns (uint256 remaining)
```

`allowance` returns the remaining quantity of tokens `_spender` is allowed to withdraw on behalf of `_owner`.

...and that's it! These nine functions are the foundation of the many thousands of ERC20 tokens on the Ethereum blockchain.
