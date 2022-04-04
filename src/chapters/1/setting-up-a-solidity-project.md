# Setting up a Solidity project

Use `forge init` to create a new project:

```bash
$ forge init tic-tac-token --template ecmendenhall/foundry-example
```

This will create a new `tic-tac-token` directory. Let's look inside:

```bash
$ cd tic-tac-token
$ tree .
.
├── foundry.toml
├── lib
│   └── ds-test
└── src
    ├── Greeter.sol
    └── test
        └── Greeter.t.sol

6 directories, 8 files
```

From the top, we have:
- `foundry.toml`, a project configuration file. 
- A `/lib` directory for project dependencies. Inside, we have one library, the [`ds-test`](https://github.com/dapphub/ds-test) unit testing framework.
- A `/src` directory for Solidity contracts. This includes:
  - `Greeter.sol`, an example contract.
  - A `/test/` directory for unit test contracts. It's conventional to give these a `.t.sol` extension to distinguish them from production code.

## The `foundry.toml` config file

Out of the box, our configuration file looks like this:

```toml
[default]
src = 'src'
out = 'out'
libs = ['lib']
verbosity = 2
```

Config options are namespaced by profiles. Here, we have a default profile named `default`, with a few options configured to tell Foundry where to look for source code and libraries, where to write compiled output, and a default verbosity level. (Note how it mirrors the directory structure of the project). There are many more [configuration options](https://github.com/gakonst/foundry/tree/master/config) available, but these defaults will do for now.
