# The ERC721 standard

Like their ERC20 predecessors, ERC721 tokens are based on a pretty simple, human readable [smart contract standard](https://eips.ethereum.org/EIPS/eip-721).

Essentially, an ERC721 token is a unique ID associated with an address, plus an optional blob of metadata describing the token. Like ERC20s, users may transfer ERC721s to other adresses directly or by approving third parties to move tokens on their behalf. As the EIP number indicates, ERC721 was created a few years after ERC20, and some of the design decisions in the standard are reactions to some of the shortcomings of the ERC20 standard.

Here's the core ERC721 interface:

```solidity
function balanceOf(address _owner) external view returns (uint256);
function ownerOf(uint256 _tokenId) external view returns (address);
function transferFrom(
    address _from,
    address _to,
    uint256 _tokenId
) external payable;
function safeTransferFrom(
    address _from,
    address _to,
    uint256 _tokenId,
    bytes data
) external payable;
function safeTransferFrom(
    address _from,
    address _to,
    uint256 _tokenId
) external payable;
function approve(address _approved, uint256 _tokenId) external payable;
function getApproved(uint256 _tokenId) external view returns (address);
function setApprovalForAll(address _operator, bool _approved) external;
function isApprovedForAll(
    address _owner,
    address _operator
) external view returns (bool);
```

You’ll notice some familiar function names shared by ERC20, like `balanceOf`, `transferFrom`, and `approve`. Rather than working with fungible `amount`s, each of these functions takes a unique `tokenId` as an argument, or returns a unique ID as a value:

```solidity
function balanceOf(address _owner) external view returns (uint256);
function transferFrom(
    address _from,
    address _to,
    uint256 _tokenId
) external payable;
function approve(address _approved, uint256 _tokenId) external payable;
```

Other functions are responses to the lessons learned from ERC20. The `safeTransferFrom` method must first check if the recipient address supports a transfer before sending a token, by calling a special `onERC721Received` function on the receiver address, if it is a contract. (This enhancement was inspired by the many ERC20 tokens that have been permanently locked in contract  addresses without any means to recover them). `getApproved`, `setApprovalForAll`, and `isApprovedForAll` adapt and extend the ERC20 concept of allowances, enabling users to allow a third party to spend _all_ tokens and adding the ability to query approved addresses.

```solidity
function safeTransferFrom(address _from, address _to, uint256 _tokenId, bytes data) external payable;
function safeTransferFrom(address _from, address _to, uint256 _tokenId) external payable;
function getApproved(uint256 _tokenId) external view returns (address);
function setApprovalForAll(address _operator, bool _approved) external;
function isApprovedForAll(address _owner, address _operator) external view returns (bool);
```

Finally, one function unique to ERC721, which returns the owner of a specific token by ID:

```jsx
function ownerOf(uint256 _tokenId) external view returns (address);
```

In addition to this core standard, ERC721 tokens may include an optional metadata interface that should look familiar. (And although this is technically optional, pretty much every token does in practice).

```jsx
function name() external view returns (string _name);
function symbol() external view returns (string _symbol);
function tokenURI(uint256 _tokenId) external view returns (string);
```

The most interesting thing here is the `tokenURI` function: this returns a URI that should point to JSON data including a name, description, and image. The standard includes a short JSON schema to describe this metadata:

```json
{
    "title": "Asset Metadata",
    "type": "object",
    "properties": {
        "name": {
            "type": "string",
            "description": "Identifies the asset to which this NFT represents"
        },
        "description": {
            "type": "string",
            "description": "Describes the asset to which this NFT represents"
        },
        "image": {
            "type": "string",
            "description": "A URI pointing to a resource with mime type image/* representing the asset to which this NFT represents. Consider making any images at a width between 320 and 1080 pixels and aspect ratio between 1.91:1 and 4:5 inclusive."
        }
    }
}
```

Exactly where and how to store this metadata is left unspecified in the spec. In practice, there are many ways to store token metadata, with different tradeoffs around durability and decentralization:

- Returning metadata from a central API (centralized, less durable)
- Returning metadata from a storage provider like S3 (centralized, more durable)
- Storing metadata on a decentralized storage network like [Filecoin](https://filecoin.io/) or [Arweave](https://www.arweave.org/) (more decentralized, potentially less durable)
- Storing metadata on a decentralized file system like [IPFS](https://ipfs.io/) (more decentralized, more low level)
- Storing metadata directly in contract storage using [data URIs](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/Data_URIs) (most decentralized, most durable, potentially very expensive)

Like ERC20s, the core behavior in ERC721 is mostly tracking balances by address. As we'll see much, of the interesting stuff lives in the token metadata.
