# Some Solidity syntax

Let's stop and cover a bit of Solidity syntax while we're here: [visibility](https://docs.soliditylang.org/en/latest/contracts.html#visibility-and-getters), [value types](https://docs.soliditylang.org/en/latest/types.html#value-types), and [reference types](https://docs.soliditylang.org/en/latest/types.html#reference-types). For reference, here's the function we just defined:

```solidity
contract FizzBuzz {
    function fizzbuzz(uint n) public returns (string memory) {
        return "fizz";
    }
}
```

#### Visibility
We've defined `fizzbuzz` as a `public` function, which means it can be called both internally by other methods in our contract and externally through message sends. There are a few other function visibility modifiers in Solidity: `external` functions can be called by other contracts but not internally, `internal` functions can only be accessed internally, and `private` functions can only be accessed internally *and* are not visible to derived contracts. Variables have `public`, `internal`, and `private` visibility, too.

#### Value types
Our `fizzbuzz` function takes one parameter `n` as an argument and returns a string. The input parameter `n` is a `uint`, or unsigned 256-bit integer. "Unsigned" means this integer type represents a non-negative integer value. Integers in Solidity are _value types_, i.e. always copied and passed by value when they are used in arguments and assignments.

256-bit integers are very large: a `uint256` can store a value as large as \\( 2^{256}-1 \\), which is way bigger than the 32-bit and 64-bit integers used by default in most other common programming languages. 

To compare the maximum value of the two types, let's jump into a Python shell and look at the difference:

```bash
$ python
>>> (2 ** 64) - 1
18446744073709551615
>>> (2 ** 256) - 1
115792089237316195423570985008687907853269984665640564039457584007913129639935
```

The `uint` type is implicitly interpreted as `uint256`, but it's considered good Solidity style to always be explicit and prefer using `uint256` to bare `uint`.

#### Reference types
The return value of our function is `(string memory)`, a _reference type_. Structs, arrays, and mappings are all reference types in Solidity. (Strings are secretly arrays of bytes under the hood, so they are reference types too). Unlike value types, which are copied each time they are used, reference types are passed by reference, so we have to be more careful about how they are used and modified to avoid unexpected mutations.

When we declare a reference type, we must also always declare the "data area" where it will be stored. There are three options: `calldata`, `memory`, and `storage`. In the case of our return value, we're using `memory`. 

`calldata` is a special, immutable, super-temporary location for function arguments. When you can get away with using it, `calldata` is a great location because it's immutable, avoids copies, and is cheap to use.

`memory` is a temporary location analogous to runtime memory. Every function call gets access to a freshly cleared chunk of memory that can expand as necessary. Writing to memory is much cheaper than writing to `storage`, but it still costs gas to read and write.

`storage` is a permanent location that is persistent between function calls. It is expensive to read and very expensive to initialize and write. (This is for good reason: any data we write to storage will be replicated on every node in the Ethereum network and stored forever!)

Onward. Let's run our tests again now that we understand our own code.
