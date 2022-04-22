# Refactoring to modifiers

To finish up, let's introduce some new Solidity syntax that will help us simplify even further. We can use [function modifiers](https://docs.soliditylang.org/en/latest/contracts.html#function-modifiers) to perform our authorization checks for the owner and player addresses.

Modifiers are sort of similar to macros in other languages, and look like this:

```solidity
modifier onlyOwner {
  require(
    msg.sender == address(0x1234),
    "only owner can call this function"
   );
  _;
}

modifier checkMoreThanFourCats {
  require(
    cats > 4,
    "cats must be greater than 4"
   );
  _;
}

modifier setFrobTo43 {
  frob = 43;
  _;
}

modifier requireBalance(uint256 amount) {
  require(balance > amount, "insufficient balance");
  _;
}

modifier nonReentrant() {
  require(!locked, "contract locked");
  locked = true;
   _;
  locked = false;
}
```

When a modifier is applied to a function, the compiler replaces the `_` placeholder inside the modifier with the body of the function it's applied to. For example, these two functions...

```solidity
function onlyOwnerCanCallThis() public onlyOwner {
  _doAdminStuff();
}

function sendTokens(address token, address recipient) public nonReentrant {
  IERC20(token).transfer(recipient);
}
```

..."expand" at compile time to the following:

```solidity
function onlyOwnerCanCallThis() public {
  require(
    msg.sender == address(0x1234),
    "only owner can call this function"
   );
  _doAdminStuff();
}

function sendTokens(address token, address recipient) public {
  require(!locked, "contract locked");
  locked = true;
  IERC20(token).transfer(recipient);
  locked = false;
}
```

Let's refactor our `_validPlayer` function and admin check to use modifiers:

```solidity
    modifier onlyOwner() {
        require(msg.sender == admin, "Unauthorized");
        _;
    }

    modifier onlyPlayer() {
        require(_validPlayer(msg.sender), "Unauthorized");
        _;
    }

    function markSpace(uint256 space) public onlyPlayer {
        require(_validTurn(msg.sender), "Not your turn");
        require(_validSpace(i), "Invalid space");
        require(_emptySpace(i), "Already marked");
        board[space] = _getSymbol(msg.sender);
        _turns++;
    }

    function reset() public onlyAdmin {
        delete board;
    }
```

We can now re-use these modifiers if we need to apply access control to other functions.

Modifiers can be very helpful, but they can also introduce indirection and complexity. It's best to keep them simple and use them sparingly.
