# Unit Testing Smart Contracts

  
  

In this tutorial, the [cw-template](https://github.com/InterWasm/cw-template) is modified to cover more unit test cases. In particular, **State.last_reset** , which contains the **timestamp** of the block when **reset** has been executed, is modified.

This tutorial will 

- structure tests,

- create a unit test,

- mock data,

- use helpers.

  

To see the structure of the code base in this tutorial, check [here](https://github.com/emidev98/counter-extended-testing/tree/main/contracts/counter).

  

## Prerequisites

This tutorial assumes that the prerequisites defined in [the `cw-template` README](https://github.com/InterWasm/cw-template#cosmwasm-starter-pack) have been met. In addition, this tutorial requires the following:

- A text editor or IDE of your choice
- A command line application 
- A fork of the [cw-template](https://github.com/InterWasm/cw-template)

## 1.1 Modify the Counter Contract

In this section, the [`cw-template`](https://github.com/InterWasm/cw-template) is modified in order to run unit tests. The following files in the `src` directory are modified:

- `msg.rs`
- `state.rs`
- `contract.rs`

Follow the steps below to modify the files and prepare for unit testing.

1. Open a terminal application.
2. Navigiate to the fork of [`cw-template`](https://github.com/InterWasm/cw-template).
3. Navigate to the `src` directory:
    ```
   cd src
   ```
4. Open `msg.rs` in a text editor.
5. Copy and paste the following code into `msg.rs`:

    ```Rust
    
    use cosmwasm_std::{Timestamp, CustomQuery};
    
    // --snip--
    
      
    
    #[derive(Serialize, Deserialize, Clone, Debug, PartialEq, JsonSchema)]
    
    #[serde(rename_all = "snake_case")]
    
    pub enum QueryMsg {
    
    // GetCount returns the current count as a json-encoded number
    
    GetCount {},
    
    // Return last reset block time
    
    GetLastResetTime { },
    
    }
    
      
    
    impl CustomQuery for QueryMsg {}
    
      
    
    // --snip--
    
      
    
    #[derive(Serialize, Deserialize, Clone, Debug, PartialEq, JsonSchema)]
    
    pub struct LastResetTime {
    
    pub last_reset_time: Option<Timestamp>,
    
    }
    
      
    
    ```

6. Close and save `msg.rs`.
7. Open `state.rs`.
8. Copy and paste the following code into `state.rs` :

    ```Rust
    
    // --snip--
    
    #[derive(Serialize, Deserialize, Clone, Debug, PartialEq, JsonSchema)]
    
    pub struct State {
    
    pub count: i32,
    
    pub owner: Addr,
    
    pub last_reset: Option<Timestamp>
    
    }
    
    ```
    
    <!--- Code blocks or admonitions between numbered steps need to be indented using 3 spaces. Do not use tab. Otherwise, the numbers will start over after the code block. -->

  
  

9. Open `contract.rs`.
10.  Copy and paste the following code into `contract.rs`:

```Rust

// --snip--

use crate::msg::{CountResponse, ExecuteMsg, InstantiateMsg, LastResetTime, QueryMsg};

// --snip--

const CONTRACT_NAME: &str = "crates.io:cw-counter";

// --snip--

#[cfg_attr(not(feature = "library"), entry_point)]

pub fn instantiate(

deps: DepsMut,

_env: Env,

info: MessageInfo,

msg: InstantiateMsg,

) -> Result<Response, ContractError> {

let state = State {

count: msg.count,

owner: info.sender.clone(),

last_reset: None,

};

set_contract_version(deps.storage, CONTRACT_NAME, CONTRACT_VERSION)?;

STATE.save(deps.storage, &state)?;

  

Ok(Response::new()

.add_attribute("method", "instantiate")

.add_attribute("owner", info.sender)

.add_attribute("count", msg.count.to_string()))

}

#[cfg_attr(not(feature = "library"), entry_point)]

pub fn execute(

deps: DepsMut,

env: Env,

info: MessageInfo,

msg: ExecuteMsg,

) -> Result<Response, ContractError> {

match msg {

ExecuteMsg::Increment {} => try_increment(deps),

ExecuteMsg::Reset { count } => try_reset(deps, env, info, count),

}

}

// --snip--

pub fn try_reset(

deps: DepsMut,

env: Env,

info: MessageInfo,

count: i32

) -> Result<Response, ContractError> {

STATE.update(deps.storage, |mut state| -> Result<_, ContractError> {

if info.sender != state.owner {

return Err(ContractError::Unauthorized {});

}

state.last_reset = Some(env.block.time);

state.count = count;

Ok(state)

})?;

Ok(Response::new().add_attribute("method", "reset"))

}

  

#[cfg_attr(not(feature = "library"), entry_point)]

pub fn query(deps: Deps, _env: Env, msg: QueryMsg) -> StdResult<Binary> {

match msg {

QueryMsg::GetCount {} => to_binary(&query_count(deps)?),

QueryMsg::GetLastResetTime { } => to_binary(&query_last_reset_tile(deps)?),

}

}

// --snip--

fn query_last_reset_tile(deps: Deps) -> StdResult<LastResetTime> {

let state = STATE.load(deps.storage)?;

Ok(LastResetTime {

last_reset_time: state.last_reset,

})

}

```

  

## 2. Create the Testing Sub-Directory

In this section, a `tests` sub-directory with various sub-modules is created to prevent code bloat. To get started, understand the structure of the `test`  directory.

### 2.1 The `test` sub-directory structure

In this section, a `test` sub-directory with the modules shown below will be created. See below to learn more about each module: 
```

├── contract.rs // Entry points of the smart contract

└── tests/ // Module that contains ONLY test-related code

├── env_mocks.rs // Example mock for the environment

├── helper.rs // Common, reusable functions to avoid unnecessary code duplication

├── mod.rs // Module entry point where tests are located

└── query_mocks.rs // Mocked QueryMsg data

```
 Now, understand how each unit test is structured.

### 2.2 Testing structure

Each unit test is structured using the [Given-When-Then](https://en.wikipedia.org/wiki/Given-When-Then) testing framework. The framework defines a simple, robust way to structure unit tests that can indicate when helper functions are needed. Each clause is defined below:

- GIVEN: The initial state and preconditions defined prior to the test.

- WHEN: Actions taken by the user during the test.

- THEN: Assertions that the actions defined in the WHEN clause executed as expected.

  

## 3. Create the helper functions

In this section, helper functions are created in `helper.rs`. Each helper function is structured using the [KISS principle](https://en.wikipedia.org/wiki/KISS_principle) to minimize complexity and code duplication. The following helper functions are created:

- `instantiate_default`
- `instantiate_with_custom_querier` 
- `custom_mock_dependencies`

To begin, create the  `instantiate_default` helper function.

### 3.1 Create `instantiate_default`

The `instantiate_default` helper function instantiates the contract with `count` set to `25` and `owner` set to `creator`. This helper function returns the contract dependencies, the message that was sent to instantiate the contract, and the response from instantiate method.

1. Open  `helpers.rs` in a text editor.
2. Copy and paste the `instantiate_default` code below into `helpers.rs`. 

```Rust

pub fn instantiate_default() -> (

OwnedDeps<MockStorage, MockApi, MockQuerier>,

MessageInfo,

Response,

) {

let mut deps = mock_dependencies(&[]);

let msg = InstantiateMsg { count: 25 };

let info = mock_info("creator", &vec![]);

let res = instantiate(deps.as_mut(), mock_env(), info.clone(), msg).unwrap();

  

(deps, info, res)

}

```
Next, create the  `instantiate_with_custom_querier` helper function.

### 3.2 Create the `instantiate_with_custom_querier` 

The `instantiate_with_custom_querier` helper function will instantiate the contract with `count` set to `25` and `owner` set to `creator`. However, unlike `instantiate_default`, this function returns the contract dependencies with MockQuerier used to mock the requests to QueryMsg, along with the message that was send to instantiate the contract and the response from instantiate method.

1. Open `helpers.rs` if it is not already open.
2. Copy and paste the `instantiate_with_custom_querier` code below into `helpers.rs`. 

```Rust

pub fn instantiate_with_custom_querier() -> (

OwnedDeps<MockStorage, MockApi, MockQuerier<QueryMsg>>,

MessageInfo,

Response,

) {

let mut deps = custom_mock_dependencies(&[]);

let msg = InstantiateMsg { count: 25 };

let info = mock_info("creator", &vec![]);

let res = instantiate(deps.as_mut(), mock_env(), info.clone(), msg).unwrap();

  

(deps, info, res)

}
```

Next, create the `custom_mock_dependencies` function.

### 3.3 Create `custom_mock_dependencies` 

The `custom_mock_dependencies` helper function is a private function that injects mocked data into `custom_query_msg`. Mocked data can be found in a different files depending on the complexity of the smart contract. The helped function returns contract dependencies with `custom_mocked_data`.

1. Open `helpers.rs` if it is not already open.
2. Copy and paste the `custom_mock_dependencies` code below into `helpers.rs`. 

```Rust

fn custom_mock_dependencies(contract_balance: &[Coin]) ->

OwnedDeps<MockStorage, MockApi, MockQuerier<QueryMsg>>

{

let custom_querier: MockQuerier<QueryMsg> =

MockQuerier::new(&[("counter_contract", contract_balance)])

.with_custom_handler(|query| SystemResult::Ok(crate::tests::query_mocks::custom_query_msg(query)));

  

OwnedDeps {

storage: MockStorage::default(),

api: MockApi::default(),

querier: custom_querier,

}

}

  

```
Next, create the mock data.

## 4. Create the mock data

  

In this section, two files containing mock data for testing are created. Each file is described below

- The `env_mocks.rs` contains mock contract and block data
- 

### 4.1 Create `env_mocks.rs`

The `env_mocks.rs` file contains mock contract and block data. This allows for testing of functionalities that depend on the block height, time or address. To create the file, complete the following steps:

1. Navigate to
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

### 4.2 Create `query_mocks.rs`

The `query_mocks.rs` file contains mock incoming query requests. This can be useful when two smart contracts are chained and only one contract needs to be tested or when validating a case that is created by multiple steps.

1. Navigate to
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

## 5. Create the unit tests

In this section, you will create the unit tests for the modified `Counter` contract. Each test will be located inside the `mod.rs` module.

:::{tip} Unit tests in production
For the purpose of this example, only one module with a few simple unit tests is created. However, in production, multiple modules may be needed for increasing test cases. depending on complexity of the contract, 
:::

### 5.0 Unit test creation prerequisites

1. Navigate to
2. Create a file named `mod.rs`


 ### 5.1 Create the `proper_initialization` test

The test validates that `instantiate_default()` returns the expected values. This test is useful for identifying initialization errors. For example, modifying data inside `instantiate_default() `can break tests.


1. Open `mod.rs`.
2. Copy and paste the `proper_initialization` code below into `mod.rs`: 

```Rust

#[test]

fn proper_initialization() {

// GIVEN properties inside instantiate_default()

  

// WHEN

let (_s, info, res) = instantiate_default();

  

// THEN

assert_eq!(

Response::new()

.add_attribute("method", "instantiate")

.add_attribute("owner", "creator")

.add_attribute("count", "25"),

res

);

assert_eq!(

MessageInfo {

sender: Addr::unchecked("creator"),

funds: vec![]

},

info

);

}

  

```

 ### 5.2 Create the `reset_with_custom_mocked_env` test

The test validates that block time is stored correctly within `custom_mocked_env()`once `ExecuteMsg:Reset` is executed.


1. Open `mod.rs`.
2. Copy and paste the `reset_with_custom_mocked_env` code below into `mod.rs`:   

```Rust

#[test]

fn reset_with_custom_mocked_env() {

// GIVEN

let (mut deps, info, _) = instantiate_default();

let msg = ExecuteMsg::Reset { count: 5 };

  

// WHEN

execute(deps.as_mut(), custom_mocked_env(), info, msg).unwrap();

let res = query(deps.as_ref(), mock_env(), QueryMsg::GetLastResetTime {}).unwrap();

let value: LastResetTime = from_binary(&res).unwrap();

  

// THEN

assert_eq!(Some(Timestamp::from_nanos(10)), value.last_reset_time);

}

```
 ### 5.2 Create the `query_mocked_last_reset_time` test

The `query_mocked_last_reset_time` test is designed to help understand how a `CustomQuerier` can be configured in order to bypass previous steps. In this test, there is no call to `Reset`. However, the `GetLastResetTime` query does not return `None`, because a mock is configured to always return the values from the method `query_mocks::custom_query_msg(...)`.


1. Open `mod.rs`.
2. Copy and paste the `reset_with_custom_mocked_env` code below into `mod.rs`:   
3. 

  

```Rust

#[test]

fn query_mocked_last_reset_time() {

// GIVEN

let (deps, _, _) = instantiate_with_custom_querier();

let req: QueryRequest<_> = QueryMsg::GetLastResetTime {}.into();

let wrapper = QuerierWrapper::new(&deps.querier);

  

// WHEN

let response: LastResetTime = wrapper.custom_query(&req).unwrap();

  

// THEN

assert_eq!(Some(Timestamp::from_nanos(420)), response.last_reset_time);

}

```

  

#### Challenge Exercise: Create a test to validate that `count` was set to `5`

Using the KISS principle, create a test to validate that `count` was set to `5`. Give it a try!


