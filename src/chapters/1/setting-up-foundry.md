# Setting up Foundry
First, install `foundryup`, the Foundry toolchain installer:

```bash
$ curl -L https://foundry.paradigm.xyz | bash
```

Inspired by the [`rustup`](https://rustup.rs/) installer for Rust, `foundryup` is a lightweight, one step installation tool. Once it's installed, reload your `PATH` or open a new terminal window and run:

```bash
$ foundryup
```

You should see output like the following in your terminal:

```bash
foundryup: installing foundry (version nightly, tag nightly-a0db055a68733f3046ca772f)
foundryup: downloading latest forge and cast
############################################# 100.0%
############################################# 100.0%
foundryup: downloading manpages
############################################# 100.0%
foundryup: installed - forge 0.2.0 (a0db055 2022-04-03T00:03:53.441110+00:00)
foundryup: installed - cast 0.2.0 (a0db055 2022-04-03T00:03:53.441110+00:00)
foundryup: done
```

To verify that Foundry is installed, run `forge --version`:

```bash
$ forge --version
forge 0.2.0 (a0db055 2022-04-03T00:03:53.441110+00:00)
```

And `cast --version`:

```bash
$ cast --version
cast 0.2.0 (a0db055 2022-04-03T00:03:53.441110+00:00)
```

Foundry is under active development, so it's a good idea to run `foundryup` regularly to install the latest changes. The Foundry team publishes new builds nightly.  

Once Foundry is installed, you should be able to interact with the `forge` and `cast` command line tools. Let's try `cast`, a multitool for interacting with Ethereum. We'll use the `--to-ascii` command, which converts a hex value to an ASCII string:

```bash
$ cast --to-ascii 0x48656c6c6f20776f726c64
Hello world
```

It works!
