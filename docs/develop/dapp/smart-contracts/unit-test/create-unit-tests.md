## 5. Create the unit tests

In this section, you will create the unit tests for the modified `Counter` contract. 
Each test will be located inside the `mod.rs` module.

:::{tip} 
**Unit tests in production**
For the purpose of this tutorial, 
one testing module with a few simple unit tests has been created. 
However, in production, multiple modules may be needed for increasing test cases, 
depending on complexity of the contract.
:::

### 5.0 Unit test creation prerequisites

1. In the `cw-template` fork, create a directory called `tests`.
2. Navigate to `tests`.
3. Create a module named `mod.rs`

 ### 5.1 Create the `proper_initialization` test

The `proper_initialization` test validates that `instantiate_default()` returns the expected values. 
This test is useful for identifying initialization errors. 
For example, modifying data inside `instantiate_default() `can break tests.
To create the test, do the following:

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

The `reset_with_custom_mocked_env` test validates that the block time is stored correctly 
within `custom_mocked_env()`once `ExecuteMsg:Reset` is executed.
To create the test, do the following:


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

The `query_mocked_last_reset_time` test is designed to 
help understand how a `CustomQuerier` can be configured in order to bypass previous steps. 
In this test, there is no call to `Reset`. 
However, the `GetLastResetTime` query does not return `None`, 
because mock data is configured to always return the values 
from the method `query_mocks::custom_query_msg(...)`.
To create this test, do the following:


1. Open `mod.rs`.
2. Copy and paste the `reset_with_custom_mocked_env` code below into `mod.rs`:   

  
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

Congratulations! You've successfully completed this tutorial. Now, try the challenge exercise below.

# Challenge Exercise: Create a test to validate that `count` was set to `5`

Using the KISS principle, create a test to validate that `count` was set to `5`. Give it a try!

