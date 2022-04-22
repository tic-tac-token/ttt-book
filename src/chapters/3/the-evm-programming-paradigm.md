# The EVM programming paradigm

Running smart contract code on a public blockchain is a very different paradigm from running code in other environments. Programming the EVM means programming a shared "world computer" with a few unusual properties. In particular, the EVM execution environment is public, permissionless, immutable, costly, deterministic, and adversarial. Let's look at each of these properties in turn.

#### Public
Smart contract code is deployed in public, replicated across thousands of nodes on the Ethereum network. Anyone can see the deployed bytecode of any contract, decompile it, and interact with it. Any legitimate project will open-source and verify their code if they expect anyone to trust it. If you like poking around with “view source” on the frontend, you’ll love this property: it’s like “view source” for smart contract code.

In addition to reading contract code, anyone on the network can view a contract's event logs and read its underlying state data from storage. (This includes internal and private storage variables, by the way!)

#### Permissionless
In addition to reading code and data, anyone on the network can call the public API of any contract at any time. It’s as if your API is always open to the world, your application logs are out in the open, and your database read replica is public. Web services kind of work this way: usually anyone can find your API endpoints and call them with unexpected arguments. But public blockchains take this property to the extreme.

#### Immutable
Once a contract is deployed, it can’t be changed. There are patterns for designing upgradeable systems and migrating from one contract to another, but ultimately the underlying code is immutable. Your bugs will be deployed forever and you have to get it right the first time. Hope you wrote some unit tests!

#### Costly
Every state changing interaction on the network has an associated cost in money, measured in units called "gas." Deploying a new contract costs gas. Calling a function on a contract costs gas. Writing data to a storage variable costs gas. Every computation and storage operation has an associated cost. There is actually a good reason for gas: it’s a defense against malicious programs that might otherwise loop forever, and serves as payment for the computation and storage you consume.

#### Deterministic
The EVM is a deterministic state machine. Since every node on the network must be able to validate any computation, every computation is deterministic. On one hand, this means common tasks like generating a random number, reading a file from the filesystem, or making an HTTP request are impossible on the EVM. On the other hand, it means we can precisely simulate the outcome of any operation starting at some known state.

#### Adversarial
Finally, all of the properties above combined with the fact that smart contracts are used to store and send value mean that even the smallest vulnerabilities will be found and exploited. In the EVM programming paradigm, it’s critical to test extensively, simulate the unexpected, and think adversarially at every turn.
