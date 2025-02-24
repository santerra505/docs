# 2. Create the helper functions

In this section, helper functions are created in `helper.rs`. Each helper function is structured using the [KISS principle](https://en.wikipedia.org/wiki/KISS_principle) to minimize complexity and code duplication. The following helper functions are created:

- `instantiate_default`
- `instantiate_with_custom_querier` 
- `custom_mock_dependencies`

To begin, create the  `instantiate_default` helper function.

## 1. Create `instantiate_default`

The `instantiate_default` helper function instantiates the contract with `count` set to `25` and `owner` set to `creator`. This helper function returns the contract dependencies, the message that was sent to instantiate the contract, and the response from instantiate method.

1. Open  `helpers.rs`.
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

## 2 Create the `instantiate_with_custom_querier` 

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

## 3. Create `custom_mock_dependencies` 

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
