# Unit Testing Smart Contracts 

In this tutorial, the [cw-template](https://github.com/InterWasm/cw-template) 
is modified to demonstrate smart contract unit test cases.
To begin, learn what the `cw-template` is,
why it is being modified, and what modifications are being made.
Then, complete the prerequisites and get started building.
 
# Overview

The [cw-template](https://github.com/InterWasm/cw-template) is a template 
for builing Rust smart contracts inside Terra and other blockchains
in the Cosmos ecosystem. 

In this tutorial, the following changes are made to `cw-template`:

1. The ***Counter*** smart contract, which is defined in the `src` directory, is modified.
2. Addtional functions are added to `helper.rs`.
3. A subdirectory `tests` is created for 
4. Mock testing data modules are created and added to `tests`.
5. Unit tests are created and added to `tests`.

In addition, a `test` subdirectory  is created to contain code for 
all unit testing capabilities. Each module or directory contained in `test`,
along with its description, is shown below:

| Module         | Module description                          |
|----------------|---------------------------------------------|
| contract.rs    | Module with smart contract entry points     |
| env_mock.rs    | Mocked environment data                     |
| mod.rs         | Defines entry points for the tests          |
| query_mocks.rs | Mocked `QueryMsg` data                      |

## Prerequisites

This tutorial assumes that the prerequisites defined in [the `cw-template` README](https://github.com/InterWasm/cw-template#cosmwasm-starter-pack) have been met. In addition, this tutorial requires the following:

> Installing Rust and `cargo` using `rustup`
>
> Using `rustup` is easiest way to install the latest versions of 
> `cargo ` and Rust. For more information, see [Cargo docs](https://doc.rust-lang.org/cargo/getting-started/installation.html).

- A code editor or IDE
- A command line application
- [`rustup`](https://rustup.rs/)
- The latest version of [Rust](https://www.rust-lang.org/tools/install) and Cargo 
- A fork of the [cw-template](https://github.com/InterWasm/cw-template)

# Getting Started

To get started,
complete each section listed below:

 ```{toctree}
 :maxdepth: 2
 modify-counter-contract
 create-helper-functions
 create-mock-testing data
 create-unit-tests
 ```
