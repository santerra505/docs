# Unit Testing Smart Contracts 

In this tutorial, the [cw-template](https://github.com/InterWasm/cw-template) 
is modified to demonstrate smart contract unit test cases.
To begin, learn what the `cw-template` is,
why it is being modified,
and then get started building.
 
# Overview

The [cw-template](https://github.com/InterWasm/cw-template) is a template 
for builing Rust smart contracts inside Terra and other blockchains
in the Cosmos ecosystem. 

In this tutorial, the following modules in the `src` directory of 
`cw-template` are modified:

- `msg.rs`
- `state.rs`
- `contract.rs`
- `helpers.rs`

In addition, a `test` subdirectory  is created to contain code for 
extended unit testing capabilities. Each module or directory contained in `test`,
along with its description, is shown below:

| contract.rs    | Module for smart contract entry points      |
|----------------|---------------------------------------------|
| contract.rs    | Module with smart contract entry points   |
| env_mock.rs    | Module with smart contract entry points                             |
| helper.rs      | Common, reusable functions                  |
| mod.rs         | Defines entry points for the tests          |
| query_mocks.rs | Mocked `QueryMsg` data                      |
| tests/         | A subdirectory for test-related code       |


# Getting Started

To get started building the CW20 Token Factory,
complete each section listed below:

 ```{toctree}
 :maxdepth: 2
 setup
 modify-cw20
 create-token-contract
 ```
