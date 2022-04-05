# Creating a Foundry project

`forge` is the Foundry command line tool for managing projects, compiling contracts, and running tests. 

Let's run `forge init` to create a new project from a template. (I've created this one with some predefined example contracts that will show us some of the features of `forge` and Solidity. To create a new project without a template, you can simply use `forge init`):

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
- A `lib/` directory. This is used for project dependencies. Inside, we have one library, the [`ds-test`](https://github.com/dapphub/ds-test) unit testing framework.
- A `src/` directory. This is used for Solidity contracts, including tests. This includes:
  - `Greeter.sol`, an example "Hello World" contract.
  - A `test/` directory for unit test contracts. It's conventional to give these a `.t.sol` extension to distinguish them from production code.

## The `foundry.toml` config file

Out of the box, our configuration file looks like this:

```toml
[default]
src = 'src'
out = 'out'
libs = ['lib']
verbosity = 2
```

Config options are namespaced by profiles. Here, we have a single default profile named `default`, with a few options configured to tell Foundry where to look for source code and libraries, where to write compiled output, and a default verbosity level. (Note how it mirrors the directory structure of the project). 

We can create additional profiles that inherit from the default and override some settings. For example, a `verbose` profile that prints more output by default:

```toml
[default]
src = 'src'
out = 'out'
libs = ['lib']
verbosity = 2

[verbose]
verbosity = 4
```

Defining profiles is a powerful way to configure Foundry for different tasks, contexts, and environments. There are many more [configuration options](https://github.com/gakonst/foundry/tree/master/config) available, but these defaults will do for now.
