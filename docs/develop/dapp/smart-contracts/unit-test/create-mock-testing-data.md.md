# 4. Create the mock data

In this section, mock data for testing is created. Each mock data module is described below:

- The `env_mocks.rs` module contains mock contract and block data.
- The `query_mocks` module contains mock incoming query request.

## 1 Create `env_mocks.rs`

The `env_mocks.rs` module contains mock contract and block data. This allows for testing of functionalities that depend on the block height, time or address. 
To create `env_mocks.rs`, complete the following steps:

1. Navigate to the `test` directory.
2. Create a file named `env_mocks.rs`
3. Copy and paste the sample data below into `env_mocks.rs`:

```Rust

pub fn custom_mocked_env() -> Env {

Env {

block: BlockInfo {

height: 420,

time: Timestamp::from_nanos(10),

chain_id: "never_went_down".to_string(),

},

contract: ContractInfo {

address: Addr::unchecked("counter_contract"),

},

}

}

```
Next, create `query_mocks.rs`.

## 2 Create `query_mocks.rs`

The `query_mocks.rs` module contains mock incoming query requests. 
This can be useful when two smart contracts are chained and only one contract needs to be tested or when validating a case that is created by multiple steps.
To create `query_mocks.rs`, complete the following steps:

1. Navigate to `test`.
2. Create a file named `query_mocks.rs`
3. Copy and paste the sample data below into `query_mocks.rs`:

```Rust

use crate::msg::{CountResponse, LastResetTime, QueryMsg};

use cosmwasm_std::{to_binary, ContractResult, QueryResponse, Timestamp};

  

pub fn custom_query_msg(query: &QueryMsg) -> ContractResult<QueryResponse> {

let msg = match query {

QueryMsg::GetCount {} => to_binary(&CountResponse { count: 69 }),

QueryMsg::GetLastResetTime {} => to_binary(&LastResetTime {

last_reset_time: Some(Timestamp::from_nanos(420)),

}),

};

msg.into()

}

```

 Next, create the unit tests.
