# Mappings

Mappings are similar to hashmaps or dictionaries: a key-value data structure that enables us to set and get values by key. To declare a mapping, we must specify its key and value type:

```solidity
mapping(address => bool) hasPlayedGame;
mapping(uint256 => address) addressById;
mapping(address => uint256[]) gamesByPlayer;
```

Mappings are a reference type, but their data location must always be `storage`. We can access a value in a mapping by key using square brackets:

```solidity
bool played = hasPlayedGame[address(0x1234)];
address account = addressById[23];
uint256[] storage games = gamesByPlayer[address(0x1234)];
```

And set them using similar syntax:

```solidity
hasPlayedGame[address(0x5678)] = true;
addressById[24] = address(0x5678);
gamesByPlayer[address(0x1234)].push(5);
```

Of course, since this is Solidity, mappings are weird in a few ways. First, keys are not stored in any way. Instead the `keccak256` hash of a key is used to look up its value. That means there's no concept of an unknown key. Second, empty values in a mapping are initialized with their default value (just like if they were declared as a state variable). When we create a new mapping, it's as if every possible key exists in the mapping and points to an empty entry with a default value.
