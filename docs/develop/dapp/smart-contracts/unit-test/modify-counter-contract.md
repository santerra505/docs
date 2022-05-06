# 1 Modify the ***Counter*** Contract

The `src` file defines a ***Counter*** smart contract with the following modules:

- `contract.rs`
- `error.rs`
- `helpers.rs`
- `integration_tests.rs`
- `lib.rs`
- `msg.rs`
- `state.rs`

In this tutorial, the following modules are modified:

- `msg.rs`
- `state.rs`
- `contract.rs`

To modify the contract, follow the steps below.

## 1. Navigate to the `src` directory

1. Open a terminal application.
2. Navigiate to the fork of [`cw-template`](https://github.com/InterWasm/cw-template).
3. Navigate to the `src` directory:
    ```
   cd src
   ```
   
## 2. Modify `msg.rs`

1. Open `msg.rs`. 
2. Above the other `use` statements, add the following:
   
   ```rust
   use cosmwasm_std::{Timestamp, CustomQuery};
   ```
   
3. In the `QueryMsg` function, do the following:
    - Return the last reset block time
    - Implement `CustomQuery`
   
   The function should now look like the following:

	```rust
	#[derive(Serialize, Deserialize, Clone, Debug, PartialEq, JsonSchema)]
	#[serde(rename_all = "snake_case")]
	pub enum QueryMsg {
	    // GetCount returns the current count as a json-encoded number
	    GetCount {},
	    // Return last reset block time
	    GetLastResetTime { },
	}
	impl CustomQuery for QueryMsg {}
	```

4. Below `CountResponse`, define the `LastResetTime` function:

   ```rust 	
   #[derive(Serialize, Deserialize, Clone, Debug, PartialEq, JsonSchema)]	
    pub struct LastResetTime {	
        pub last_reset_time: Option<Timestamp>,	
   }
   ```

5. The `msg.rs` module should now look like the following:

	```rust
	use cosmwasm_std::{Timestamp, CustomQuery};
	use schemars::JsonSchema;
	use serde::{Deserialize, Serialize};

	#[derive(Serialize, Deserialize, Clone, Debug, PartialEq, JsonSchema)]
	pub struct InstantiateMsg {
	    pub count: i32,
	}

	#[derive(Serialize, Deserialize, Clone, Debug, PartialEq, JsonSchema)]
	#[serde(rename_all = "snake_case")]
	pub enum ExecuteMsg {
	    Increment {},
	    Reset { count: i32 },
	}

	#[derive(Serialize, Deserialize, Clone, Debug, PartialEq, JsonSchema)]
	#[serde(rename_all = "snake_case")]
	pub enum QueryMsg {
	    // GetCount returns the current count as a json-encoded number
	    GetCount {},
	    // Return last reset block time
	    GetLastResetTime { },
	}

	impl CustomQuery for QueryMsg {}

	// We define a custom struct for each query response
	#[derive(Serialize, Deserialize, Clone, Debug, PartialEq, JsonSchema)]
	pub struct CountResponse {
	    pub count: i32,
	}

	#[derive(Serialize, Deserialize, Clone, Debug, PartialEq, JsonSchema)]
	pub struct LastResetTime {
	    pub last_reset_time: Option<Timestamp>,
	}
	```

6. Close and save `msg.rs`.

## 3. Modify `state.rs`

1. Open `state.rs`.
2. On line 4, add `Timestamp`. Line 4 should now look like the following:
   ```Rust
   use cosmwasm_std::{Addr, Timestamp};
   ```
3. Below `owner` in `State`, add `last_reset`:
   ```Rust
   pub last_reset: Option<Timestamp>
   ```

4. The `state.rs` module should now look like the following:

    ```Rust
    
    use schemars::JsonSchema;
	use serde::{Deserialize, Serialize};

	use cosmwasm_std::{Addr, Timestamp};
	use cw_storage_plus::Item;

	#[derive(Serialize, Deserialize, Clone, Debug, PartialEq, JsonSchema)]
	pub struct State {
		pub count: i32,
		pub owner: Addr,
		pub last_reset: Option<Timestamp>
	}

	pub const STATE: Item<State> = Item::new("state");
    
    ```

5. Save and close `state.rs`.

## 4. Modify `contract.rs`

1. Open `contract.rs`.
2. Import `LastResetTime` from the `msg` crate:

	```Rust
	use crate::msg::{CountResponse, ExecuteMsg, InstantiateMsg, LastResetTime, QueryMsg};
	```

3. Set the contract name:

   ```rust
   const CONTRACT_NAME: &str = "crates.io:cw-counter";
   ```

4. In the `instantiate` function below `owner`, set `last_reset` to `None`:

   ```rust
   last_reset: None,
   ```

5. In the `execute` function, do the following:

    - Change `env` to `_env`. 
    - In `execute` in the `match msg` block, pass `env`  to `try_reset()`.
    
    The `execute` function should now look like the following:

	```rust
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
	```
	
6. In the `try_reset` function, do the following:
	
	- Set the `env` to `Env`
	- Set the `state.last_reset` time to `Some(env.block.time)`;

    The `try_reset` function should now look like the following:
 
	```Rust
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
	```
7. Add `GetLastResetTime` to the `query` function. The function should look like the following:
	```Rust
	#[cfg_attr(not(feature = "library"), entry_point)]	
	pub fn query(deps: Deps, _env: Env, msg: QueryMsg) -> StdResult<Binary> {	
	    match msg {	
	        QueryMsg::GetCount {} => to_binary(&query_count(deps)?),	
	        QueryMsg::GetLastResetTime { } => to_binary(&query_last_reset_tile(deps)?),	
	    }	
	}
	```
8. The `contract.rs` module should now look like the following:

	```Rust
	#[cfg(not(feature = "library"))]	
	use cosmwasm_std::entry_point;	
	use cosmwasm_std::{to_binary, Binary, Deps, DepsMut, Env, MessageInfo, Response, StdResult};	
	use cw2::set_contract_version;	
	use crate::error::ContractError;	
	use crate::msg::{CountResponse, ExecuteMsg, InstantiateMsg, LastResetTime, QueryMsg};	
	use crate::state::{State, STATE};	
	// version info for migration info	
	const CONTRACT_NAME: &str = "crates.io:cw-counter";	
	const CONTRACT_VERSION: &str = env!("CARGO_PKG_VERSION");	
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
	pub fn try_increment(deps: DepsMut) -> Result<Response, ContractError> {	
	    STATE.update(deps.storage, |mut state| -> Result<_, ContractError> {	
	        state.count += 1;	
	        Ok(state)	
	    })?;	
	    Ok(Response::new().add_attribute("method", "try_increment"))	
	}	
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
	fn query_count(deps: Deps) -> StdResult<CountResponse> {	
	    let state = STATE.load(deps.storage)?;	
	    Ok(CountResponse { count: state.count })	
	}	
	fn query_last_reset_tile(deps: Deps) -> StdResult<LastResetTime> {	
	    let state = STATE.load(deps.storage)?;	
	    Ok(LastResetTime {	
	        last_reset_time: state.last_reset,	
	    })	
	}	
	#[cfg(test)]	
	mod tests {	
	    use super::*;	
	    use cosmwasm_std::testing::{mock_dependencies_with_balance, mock_env, mock_info};	
	    use cosmwasm_std::{coins, from_binary};	
	    #[test]	
	    fn proper_initialization() {	
	        let mut deps = mock_dependencies_with_balance(&coins(2, "token"));	
	        let msg = InstantiateMsg { count: 17 };	
	        let info = mock_info("creator", &coins(1000, "earth"));	
	        // we can just call .unwrap() to assert this was a success	
	        let res = instantiate(deps.as_mut(), mock_env(), info, msg).unwrap();	
	        assert_eq!(0, res.messages.len());	
	        // it worked, let's query the state	
	        let res = query(deps.as_ref(), mock_env(), QueryMsg::GetCount {}).unwrap();	
	        let value: CountResponse = from_binary(&res).unwrap();	
	        assert_eq!(17, value.count);	
	    }	
	    #[test]	
	    fn increment() {	
	        let mut deps = mock_dependencies_with_balance(&coins(2, "token"));	
	        let msg = InstantiateMsg { count: 17 };	
	        let info = mock_info("creator", &coins(2, "token"));	
	        let _res = instantiate(deps.as_mut(), mock_env(), info, msg).unwrap();	
	        // beneficiary can release it	
	        let info = mock_info("anyone", &coins(2, "token"));	
	        let msg = ExecuteMsg::Increment {};	
	        let _res = execute(deps.as_mut(), mock_env(), info, msg).unwrap();	
	        // should increase counter by 1	
	        let res = query(deps.as_ref(), mock_env(), QueryMsg::GetCount {}).unwrap();	
	        let value: CountResponse = from_binary(&res).unwrap();	
	        assert_eq!(18, value.count);	
	    }	
	    #[test]	
	    fn reset() {	
	        let mut deps = mock_dependencies_with_balance(&coins(2, "token"));	
	        let msg = InstantiateMsg { count: 17 };	
	        let info = mock_info("creator", &coins(2, "token"));	
	        let _res = instantiate(deps.as_mut(), mock_env(), info, msg).unwrap();	
	        // beneficiary can release it	
	        let unauth_info = mock_info("anyone", &coins(2, "token"));	
	        let msg = ExecuteMsg::Reset { count: 5 };	
	        let res = execute(deps.as_mut(), mock_env(), unauth_info, msg);	
	        match res {	
	            Err(ContractError::Unauthorized {}) => {}	
	            _ => panic!("Must return unauthorized error"),	
	        }	
	        // only the original creator can reset the counter	
	        let auth_info = mock_info("creator", &coins(2, "token"));	
	        let msg = ExecuteMsg::Reset { count: 5 };	
	        let _res = execute(deps.as_mut(), mock_env(), auth_info, msg).unwrap();	
	        // should now be 5	
	        let res = query(deps.as_ref(), mock_env(), QueryMsg::GetCount {}).unwrap();	
	        let value: CountResponse = from_binary(&res).unwrap();	
	        assert_eq!(5, value.count);	
	    }	
	}
	```     
