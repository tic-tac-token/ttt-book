# Exploring an implementation

The ERC20 standard defines the external interface and expected behavior of a token contract, but not the internal implementation details, like _how_ to track account balances and approvals. To see how an ERC20 token is actually implemented, let's take a line by line look at the [OpenZeppelin ERC20](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol) contract, one widely used ERC20 implementation.

Here are the first few lines:

```solidity
pragma solidity ^0.8.0;

import "./IERC20.sol";
import "./extensions/IERC20Metadata.sol";
import "../../utils/Context.sol";

contract ERC20 is Context, IERC20, IERC20Metadata {
    mapping(address => uint256) private _balances;

    mapping(address => mapping(address => uint256)) private _allowances;

    uint256 private _totalSupply;

    string private _name;
    string private _symbol;

    constructor(string memory name_, string memory symbol_) {
        _name = name_;
        _symbol = symbol_;
    }
```

We're inheriting from multiple base contracts, but don't worry too much about their details. `Context` defines wrapped helper functions for accessing `msg.sender` and `msg.data`. `IERC20` and `IERC20Metadata` are empty interfaces. These are good practices for a contract that will be inherited by others like OpenZeppelin, but not too relevant to understanding the internals of how our token implementation works.

The storage variables are more interesting: `_balances` is a mapping from address to amount, and we've defined strings to store `_name` and `_symbol` and a uint for `_totalSupply`. `_allowances` is a _nested_ mapping—its key is an address, and value is another mapping. We'll see how it's used to store allowances shortly.

The token name and symbol must be passed to the constructor and are set at construction time.

Moving on, we see implementations of the metadata functions:

```solidity
    function name() public view virtual override returns (string memory) {
        return _name;
    }

    function symbol() public view virtual override returns (string memory) {
        return _symbol;
    }

    function decimals() public view virtual override returns (uint8) {
        return 18;
    }

    function totalSupply() public view virtual override returns (uint256) {
        return _totalSupply;
    }
```

These are all wrappers around private state variables, with the exception of `decimals`, which hardcodes the conventional default of 18. (Derived contracts can override this function if they really want to use a different number of decimals, but it's generally considered good practice to stick with 18).

Next up are views for `balanceOf` and `allowance`:

```solidity
    function balanceOf(address account) public view virtual override returns (uint256) {
        return _balances[account];
    }

    function allowance(address owner, address spender) public view virtual override returns (uint256) {
        return _allowances[owner][spender];
    }
```

You can see that `balanceOf` reads the balance by address from the `_balances` mapping—pretty much the same as the points mapping in our Tic-Tac-Toe game! The `allowance` function demonstrates accessing a nested mapping—we first look up the owner's allowances mapping by address, then look up the spender's allowance from the nested `(address => uint256)` mapping.

On to token transfers, our first state-changing behavior...

```solidity
    function transfer(address recipient, uint256 amount) public virtual override returns (bool) {
        _transfer(_msgSender(), recipient, amount);
        return true;
    }

    function _transfer(
        address sender,
        address recipient,
        uint256 amount
    ) internal virtual {
        require(sender != address(0), "ERC20: transfer from the zero address");
        require(recipient != address(0), "ERC20: transfer to the zero address");

        _beforeTokenTransfer(sender, recipient, amount);

        uint256 senderBalance = _balances[sender];
        require(senderBalance >= amount, "ERC20: transfer amount exceeds balance");
        unchecked {
            _balances[sender] = senderBalance - amount;
        }
        _balances[recipient] += amount;

        emit Transfer(sender, recipient, amount);

        _afterTokenTransfer(sender, recipient, amount);
    }

    function _beforeTokenTransfer(
        address from,
        address to,
        uint256 amount
    ) internal virtual {}

    function _afterTokenTransfer(
        address from,
        address to,
        uint256 amount
    ) internal virtual {}
```

`transfer` itself is pretty boring—it passes arguments through to an internal `_transfer` helper and returns `true`. (Note how it's making a call to `_msgSender()` in the first argument to `_transfer`. This is an internal helper function derived from `Context` that simply returns `msg.sender`).

The `_transfer` helper is where the real behavior lives. In order, we have:

1. Two `require` statements checking that sender and recipient are valid addresses.
2. A call to a `_beforeTokenTransfer` hook. You can see that this is defined as an empty virtual function below. If a derived contract wants to perform some action before token transfers, they can override this virtual function to add a "before hook" to the transfer.
3. Reading the sender's balance from the `_balances` mapping and storing it in a temporary variable to be reused.
4. Another `require` statement ensuring that amount to send does not exceed the sender's balance.
5. Deducting the transfer amount from the sender's balance and adding it to the recipient's balance. The subtraction here is wrapped in an `unchecked` block, which saves some gas by skipping checks for arithmetic overflows and underflows. It's safe in this case because we've already checked `senderBalance >= amount`, so an underflow isn't possible.
6. Emitting a `Transfer` event. (We haven't covered this syntax yet, but it's kind of like a log statement).
7. An `_afterTokenTransfer` hook similar to the before hook.

Approvals follow a similar pattern as transfers, delegating to an internal helper:

```solidity
    function approve(address spender, uint256 amount) public virtual override returns (bool) {
        _approve(_msgSender(), spender, amount);
        return true;
    }

    function _approve(
        address owner,
        address spender,
        uint256 amount
    ) internal virtual {
        require(owner != address(0), "ERC20: approve from the zero address");
        require(spender != address(0), "ERC20: approve to the zero address");

        _allowances[owner][spender] = amount;
        emit Approval(owner, spender, amount);
    }
```

Short and simple: two `require` checks, setting the approval amount in the nested `_allowances` mapping, and emitting an event.

In addition to `approve`, OpenZeppelin's ERC20 defines `increaseApproval` and `decreaseApproval` functions that are not defined in the ERC20 standard:

```solidity
    function increaseAllowance(address spender, uint256 addedValue) public virtual returns (bool) {
        _approve(_msgSender(), spender, _allowances[_msgSender()][spender] + addedValue);
        return true;
    }

    function decreaseAllowance(address spender, uint256 subtractedValue) public virtual returns (bool) {
        uint256 currentAllowance = _allowances[_msgSender()][spender];
        require(currentAllowance >= subtractedValue, "ERC20: decreased allowance below zero");
        unchecked {
            _approve(_msgSender(), spender, currentAllowance - subtractedValue);
        }

        return true;
    }
```

`transferFrom` checks the sender's allowance, makes a transfer, and updates the approved allowance:

```solidity
    function transferFrom(
        address sender,
        address recipient,
        uint256 amount
    ) public virtual override returns (bool) {
        _transfer(sender, recipient, amount);

        uint256 currentAllowance = _allowances[sender][_msgSender()];
        require(currentAllowance >= amount, "ERC20: transfer amount exceeds allowance");
        unchecked {
            _approve(sender, _msgSender(), currentAllowance - amount);
        }

        return true;
    }
```

Finally, internal `_mint` and `_burn` functions to create and destroy tokens. These update both the `_balances` mapping and `_totalSupply` of the token. Note that these are *not* public functions—they are meant to be used internally according to whatever minting/burning logic is necessary in the derived contract.

```solidity
    function _mint(address account, uint256 amount) internal virtual {
        require(account != address(0), "ERC20: mint to the zero address");

        _beforeTokenTransfer(address(0), account, amount);

        _totalSupply += amount;
        _balances[account] += amount;
        emit Transfer(address(0), account, amount);

        _afterTokenTransfer(address(0), account, amount);
    }

    function _burn(address account, uint256 amount) internal virtual {
        require(account != address(0), "ERC20: burn from the zero address");

        _beforeTokenTransfer(account, address(0), amount);

        uint256 accountBalance = _balances[account];
        require(accountBalance >= amount, "ERC20: burn amount exceeds balance");
        unchecked {
            _balances[account] = accountBalance - amount;
        }
        _totalSupply -= amount;

        emit Transfer(account, address(0), amount);

        _afterTokenTransfer(account, address(0), amount);
    }
```

...and that's it again! A little over 150 lines of Solidity for an ERC20 implementation.

Comparing different implementations and their design trade-offs is an interesting exercise. If you're interested in exploring further, here are a few alternative ERC20 contracts:

- [Solmate](https://github.com/Rari-Capital/solmate/blob/main/src/tokens/ERC20.sol), an opinionated, gas optimized ERC20.
- [ds-token](https://github.com/dapphub/ds-token/blob/master/src/token.sol), a simple contract written in dapphub style.
- [Consensys](https://github.com/ConsenSys/Tokens/blob/master/contracts/eip20/EIP20.sol), an old but concise implementation.
- [MiniMeToken](https://github.com/Giveth/minime/blob/master/contracts/MiniMeToken.sol), an ERC20 with extra features for checkpointing balances and cloning tokens.
