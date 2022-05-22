# Ownership and minting

Just like our ERC20, we'll need to add the ability to mint new tokens and limit access to authorized callers.

Let's start by adding a `mint` function. We'll mint token #1 to the test contract address, then call two ERC721 methods to make sure it was created. `balanceOf` should return the total number of tokens held by the test contract, while `ownerOf(1)` should return the test contract address:

```solidity
    function test_token_is_mintable() public {
        nft.mint(address(this), 1);

        assertEq(nft.balanceOf(address(this)), 1);
        assertEq(nft.ownerOf(1), address(this));
    }
```

Let's add a `mint` function to our NFT contract to pass this test. We can use the internal `_safeMint` [function](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721-_safeMint-address-uint256) from OpenZeppelin ERC721 to mint a token with ID `tokenId` and transfer it to address `to`:

```solidity
    function mint(address to, uint256 tokenId) public {
      _safeMint(to, tokenId);
    }
```

> **Safe mints and transfers**
>
> ERC721 includes both `transferFrom` and `safeTransferFrom` functions in its public interface, and OpenZeppelin's ERC721 base contract includes internal `_mint` and `_safeMint` functions.
>
> The "safe" versions of these functions were introduced in reaction to a common usability problem with ERC20 tokens: users accidentally transferring tokens into smart contract addresses that have no mechanism for transferring them back out.
>
> To prevent the same issue with ERC721 tokens, the standard requires that `safeTransfer` must call a special `onERC721Received` function on the receiver address if it is a contract. If the receiver returns a magic 4 byte value from this function, it signals that it is able to receive ERC721 tokens. You can think of this as a callback before the token is transferred.
>
> Although this is safer when it comes to preventing accidental transfers, any call to an external contract creates a potential attack vector for [reentrancy](https://consensys.github.io/smart-contract-best-practices/attacks/reentrancy/) and other malicious behavior. Many NFT contracts have [introduced vulnerabilities](https://samczsun.com/the-dangers-of-surprising-code/) by using `safeTransfer` without protecting against reentrancy. We'll cover reentrancy in more depth later, but for now, be aware that `safeTransfer` and `_safeMint` are not always as safe as you might think!

We've added a `mint` function, now let's run the tests:

```bash
$ forge test --match-path src/test/NFT.t.sol
Compiler run successful

Running 3 tests for src/test/NFT.t.sol:TicTacTokenTest
[FAIL. Reason: ERC721: transfer to non ERC721Receiver implementer] test_token_is_mintable() (gas: 53501)
[PASS] test_token_name() (gas: 9801)
[PASS] test_token_symbol() (gas: 9775)
Test result: FAILED. 2 passed; 1 failed; finished in 2.08ms

Failed tests:
[FAIL. Reason: ERC721: transfer to non ERC721Receiver implementer] test_token_is_mintable() (gas: 53501)
```

Our tests failed, but for a good reason: `_safeMint` is working as it should! It made a callback to our test contract and reverted the transfer since it doesn't expose an `onERC721Received` function.

Let's signal that our test contract supports ERC721 transfers by implementing `onERC721Received`. We need to implement the interface described [here](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721Receiver), by adding a function to our test contract. We'll ignore all the arguments and simply return the `onERC721Received` [function selector](https://docs.soliditylang.org/en/latest/abi-spec.html#function-selector), which is the 4 byte magic value required by the spec.

```solidity
    function onERC721Received(
        address operator,
        address from,
        uint256 tokenId,
        bytes calldata data
    ) public returns (bytes4) {
        return this.onERC721Received.selector;
    }
```

Once we've made this change, our tests should pass:

```bash
Running 3 tests for src/test/NFT.t.sol:TicTacTokenTest
[PASS] test_token_is_mintable() (gas: 56503)
[PASS] test_token_name() (gas: 9779)
[PASS] test_token_symbol() (gas: 9753)
Test result: ok. 3 passed; 0 failed; finished in 4.31ms
```

We added our own `onERC721Received`, but OpenZeppelin includes an `ERC721Holder` [helper contract](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#ERC721Holder) that provides this method. Let's use it instead and remove our custom function. All we need to do is import it and inherit in our test contract:

```solidity
import "ds-test/test.sol";
import "forge-std/Vm.sol";

import "openzeppelin-contracts/contracts/token/ERC721/utils/ERC721Holder.sol";
import "../NFT.sol";

contract TicTacTokenTest is DSTest, ERC721Holder {
    ...
}
```

Finally, let's restrict minting to authorized addresses. Once again, we'll use `Ownable`:

```solidity
    function test_token_is_not_mintable_by_nonowner() public {
        vm.prank(address(1));
        vm.expectRevert("Ownable: caller is not the owner");
        nft.mint(address(this), 1);
    }
```

Let's import and add `Ownable` to our NFT contract:

```solidity
import "openzeppelin-contracts/contracts/token/ERC721/ERC721.sol";
import "openzeppelin-contracts/contracts/access/Ownable.sol";

contract NFT is ERC721, Ownable {

    constructor() ERC721("Tic Tac Token NFT", "TTT NFT") {}

    function mint(address to, uint256 tokenId) onlyOwner public {
      _safeMint(to, tokenId);
    }

}
```

And make sure it works as expected:

```
$ forge test --match-path src/test/NFT.t.sol
Running 4 tests for src/test/NFT.t.sol:TicTacTokenTest
[PASS] test_token_is_mintable() (gas: 58902)
[PASS] test_token_is_not_mintable_by_nonowner() (gas: 13279)
[PASS] test_token_name() (gas: 9807)
[PASS] test_token_symbol() (gas: 9803)
Test result: ok. 4 passed; 0 failed; finished in 1.15ms
```

The framework for our NFT is complete. Next up, we'll integrate it with the game.
