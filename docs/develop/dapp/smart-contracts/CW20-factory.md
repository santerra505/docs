# Build a CW20 tokens factory  

The scope of this guide is to create a new smart contract and adapt the fronted to be able to read and write data to the previously mentioned smart contract minting and burning CW20 Tokens.

## Prerequisites

Tools that will be used: 
- [Terrain](https://github.com/terra-money/terrain) as development environment, 
- [LocalTerra](https://github.com/terra-money/localterra) as local terra node,
- [Terra Station Desktop](https://docs.terra.money/docs/learn/terra-station/download/terra-station-desktop.html) as wallet to interact with the smart contract locally.

If the tools are not installed in your computer follow the next tutorials to install the tools:
- [Terrain initial setup](https://docs.terra.money/docs/develop/dapp/quick-start/initial-setup.html#initial-setup),
- [Install and run LocalTerra](https://docs.terra.money/docs/develop/dapp/quick-start/using-terrain-localterra.html#install-and-run-localterra),
- [Install the Terra Station Desktop](https://docs.terra.money/docs/learn/terra-station/download/terra-station-desktop.html).

:::{tip}
Import the passphrase listed below into the Terra Station Desktop [because the first generate account has already funds if you're using LocalTerra](https://github.com/terra-money/LocalTerra/blob/main/terracore/mnemonics.json#L9).

notice oak worry limit wrap speak medal online prefer cluster roof addict wrist behave treat actual wasp year salad speed social layer crew genius
:::

## Overview

This guide will allow you to:
- use terrain to instantiate a new app,
- modify a smart contract to allows minting [CW20 Tokens](https://github.com/CosmWasm/cw-plus/tree/main/packages/cw20),
- bound 1 UST with each unit of a CW20 Token minted,
- display total amount of UST stored into the smart contract.
- display minted CW20 Tokens with this contract,
- display burned CW20 Tokens with this contract,
- write tests for the smart contract.

# 1.Instantiate and configure a new app 

Assuming the prerequisites are met a new app can be created using the command below:
```sh
> ~/Documents/github$ terrain new tokens-factory
generating: 
- contract... done
- frontend... done
```

The next step is to use an IDE to open the project on my case I will use Code:
```sh
> ~/Documents/github$ cd tokens-factory/
> ~/Documents/github/tokens-factory$ code .
```

Using the terminal lets create two contracts:
```sh
> ~/Documents/github/tokens-factory$ terrain code:new tokens-factory
generating contract... done
> ~/Documents/github/tokens-factory$ terrain code:new token
generating contract... done
```

:::{tip}
Consider deleting the counter smart contract folder to have a clean workspace:
```
> ~/Documents/github/tokens-factory$ cd contracts/
> ~/Documents/github/tokens-factory/contracts$ ls
counter  tokens-factory
> ~/Documents/github/tokens-factory/contracts$ rm -r counter/
```
:::

Two libraries needs to be added to the token smart contract, the first one is [cw20](https://github.com/CosmWasm/cw-plus/tree/main/packages/cw20) which contains the fungible tokens specifications and [cw20-base](https://github.com/CosmWasm/cw-plus/tree/main/contracts/cw20-base) which contains the implementations. Lets add the package to the token contract:

> /tokens-factory/contracts/token/Cargo.toml
```toml
# ...

[dependencies]
cw20 = "0.8.1"
cw20-base = {  version = "0.8.1", features = ["library"] }

# ...
```

Before testing the smart contracts deployments lets modify the passphrase that will be used to do the deployments to LocalTerra:

> /tokens-factory/keys.terrain.js
```js
module.exports = {
  test: {
    mnemonic:
      "notice oak worry limit wrap speak medal online prefer cluster roof addict wrist behave treat actual wasp year salad speed social layer crew genius",
  }
};
```

Lets deploy each smart contract to validate that the development environment is configured correctly. As is the first time to download dependencies, compile and deploy it may take few seconds (at the end the console must should some addresses displayed):
```
> ~/Documents/github/tokens-factory$ terrain deploy tokens-factory --signer test
> ~/Documents/github/tokens-factory$ terrain deploy token --signer test
```

# 2.Token Smart Contract

Token is the smart contract that will extend [cw20_base](https://github.com/CosmWasm/cw-plus/tree/main/contracts/cw20-base) adding few modifications allowing to change the minter of the contract. Said that lets modify the content to make the contract compatible with CW20 standard:

> /tokens-factory/contracts/token/src/msg.rs
```Rust
use schemars::JsonSchema;
use serde::{Deserialize, Serialize};

use cosmwasm_std::{Binary, Uint128};
use cw20::{Expiration, Logo};

#[derive(Serialize, Deserialize, Clone, Debug, PartialEq, JsonSchema)]
#[serde(rename_all = "snake_case")]
pub enum ExecuteMsg {
    Mint { recipient: String, amount: Uint128 },
    UpdateMinter { address: String },

    // Implements CW20. Transfer is a base message to move tokens to another account without triggering actions
    Transfer { recipient: String, amount: Uint128 },

    // Burn is a message to destroy tokens forever
    Burn { amount: Uint128 },
    
    /* Implements CW20. Send is a base message to transfer tokens to a contract and trigger an action
    on the receiving contract.*/
    Send {
        contract: String,
        amount: Uint128,
        msg: Binary,
    },
    /* Implements CW20 "approval" extension. Allows spender to access an additional amount tokens
    from the owner's (env.sender) account. If expires is Some(), overwrites current allowance
    expiration with this one. */
    IncreaseAllowance {
        spender: String,
        amount: Uint128,
        expires: Option<Expiration>,
    },
    /* Implements CW20 "approval" extension. Lowers the spender's access of tokens
    from the owner's (env.sender) account by amount. If expires is Some(), overwrites current
    allowance expiration with this one. */
    DecreaseAllowance {
        spender: String,
        amount: Uint128,
        expires: Option<Expiration>,
    },
    /* Implements CW20 "approval" extension. Transfers amount tokens from owner -> recipient
    if `env.sender` has sufficient pre-approval. */
    TransferFrom {
        owner: String,
        recipient: String,
        amount: Uint128,
    },
    /* Implements CW20 "approval" extension. Sends amount tokens from owner -> contract
    if `env.sender` has sufficient pre-approval. */
    SendFrom {
        owner: String,
        contract: String,
        amount: Uint128,
        msg: Binary,
    },
    /*Only with the "marketing" extension. If authorized, updates marketing metadata.
    Setting None/null for any of these will leave it unchanged.
    Setting Some("") will clear this field on the contract storage. */
    UpdateMarketing {
        // A URL pointing to the project behind this token.
        project: Option<String>,
        // A longer description of the token and it's utility. Designed for tooltips or such
        description: Option<String>,
        // The address (if any) who can update this data structure
        marketing: Option<String>,
    },

    /// If set as the "marketing" role on the contract, upload a new URL, SVG, or PNG for the token
    UploadLogo(Logo),
}

#[derive(Serialize, Deserialize, Clone, Debug, PartialEq, JsonSchema)]
#[serde(rename_all = "snake_case")]
pub enum QueryMsg {
    // Implements CW20. Returns the current balance of the given address, 0 if unset.
    Balance { address: String },

    // Implements CW20. Returns metadata on the contract - name, decimals, supply, etc.
    TokenInfo {},
    
    /* Implements CW20 "allowance" extension. Returns how much 
    spender can use from owner account, 0 if unset. */
    Allowance { owner: String, spender: String },
}

#[derive(Serialize, Deserialize, Clone, Debug, PartialEq, JsonSchema)]
pub struct MigrateMsg {}
```

> /tokens-factory/contracts/token/src/error.rs
```Rust
use cosmwasm_std::StdError;
use thiserror::Error;

#[derive(Error, Debug, PartialEq)]
pub enum ContractError {
    #[error("{0}")]
    Std(#[from] StdError),

    #[error("Unauthorized")]
    Unauthorized {},

    #[error("CannotSetOwnAccount")]
    CannotSetOwnAccount {},

    #[error("InvalidZeroAmount")]
    InvalidZeroAmount {},

    #[error("Expired")]
    Expired {},

    #[error("NoAllowance")]
    NoAllowance {},

    #[error("CannotExceedCap")]
    CannotExceedCap {},

    #[error("LogoTooBig")]
    LogoTooBig {},
    
    #[error("InvalidXmlPreamble")]
    InvalidXmlPreamble {},
    
    #[error("InvalidPngHeader")]
    InvalidPngHeader {},
}

impl From<cw20_base::ContractError> for ContractError {
    fn from(err: cw20_base::ContractError) -> Self {
        match err {
            cw20_base::ContractError::Std(error) => ContractError::Std(error),
            cw20_base::ContractError::Unauthorized {} => ContractError::Unauthorized {},
            cw20_base::ContractError::CannotSetOwnAccount {} => {
                ContractError::CannotSetOwnAccount {}
            }
            cw20_base::ContractError::InvalidZeroAmount {} => ContractError::InvalidZeroAmount {},
            cw20_base::ContractError::Expired {} => ContractError::Expired {},
            cw20_base::ContractError::NoAllowance {} => ContractError::NoAllowance {},
            cw20_base::ContractError::CannotExceedCap {} => ContractError::CannotExceedCap {},


            cw20_base::ContractError::LogoTooBig {} => ContractError::LogoTooBig {},
            cw20_base::ContractError::InvalidXmlPreamble {} => ContractError::InvalidXmlPreamble {},
            cw20_base::ContractError::InvalidPngHeader {} => ContractError::InvalidPngHeader {}
        }
    }
}
```

> /tokens-factory/contracts/token/src/lib.rs
```Rust
pub mod contract;
pub mod msg;
mod error;
mod test;
pub use crate::error::ContractError;
```

> /tokens-factory/contracts/token/src/contract.rs
```Rust
#[cfg(not(feature = "library"))]
use cosmwasm_std::entry_point;
use cosmwasm_std::{
    to_binary, Binary, Deps, DepsMut, Env, MessageInfo, Response, StdResult
};

use cw2::set_contract_version;
use cw20_base::allowances::{
    execute_decrease_allowance, execute_increase_allowance, execute_send_from,
    execute_transfer_from, query_allowance,
};
use cw20_base::contract::{
    execute_burn, execute_mint, execute_send, execute_transfer, execute_update_marketing,
    execute_upload_logo, query_balance, query_token_info,
};
use cw20_base::msg::InstantiateMsg;
use cw20_base::state::{MinterData, TOKEN_INFO};

use crate::error::ContractError;
use crate::msg::{ExecuteMsg, MigrateMsg, QueryMsg};

// version info for migration info
const CONTRACT_NAME: &str = "crates.io:token";
const CONTRACT_VERSION: &str = env!("CARGO_PKG_VERSION");

#[cfg_attr(not(feature = "library"), entry_point)]
pub fn instantiate(
    deps: DepsMut,
    env: Env,
    info: MessageInfo,
    msg: InstantiateMsg,
) -> Result<Response, ContractError> {
    set_contract_version(deps.storage, CONTRACT_NAME, CONTRACT_VERSION)?;

    Ok(
        cw20_base::contract::instantiate(deps, env, info, msg)?
    )
}

#[cfg_attr(not(feature = "library"), entry_point)]
pub fn execute(
    deps: DepsMut,
    env: Env,
    info: MessageInfo,
    msg: ExecuteMsg,
) -> Result<Response, ContractError> {
    match msg {
        ExecuteMsg::UpdateMinter { address } => Ok(update_minter(deps, info, address)?),
        /* Default methods from CW20 Standard, for more information check the
         * following link https://github.com/CosmWasm/cw-plus/tree/main/packages/cw20 */
        ExecuteMsg::Mint { recipient, amount } => {
            Ok(execute_mint(deps, env, info, recipient, amount)?)
        }
        ExecuteMsg::Transfer { recipient, amount } => {
            Ok(execute_transfer(deps, env, info, recipient, amount)?)
        }
        ExecuteMsg::Burn { amount } => Ok(execute_burn(deps, env, info, amount)?),
        ExecuteMsg::Send {
            contract,
            amount,
            msg,
        } => Ok(execute_send(deps, env, info, contract, amount, msg)?),
        ExecuteMsg::IncreaseAllowance {
            spender,
            amount,
            expires,
        } => Ok(execute_increase_allowance(
            deps, env, info, spender, amount, expires,
        )?),
        ExecuteMsg::DecreaseAllowance {
            spender,
            amount,
            expires,
        } => Ok(execute_decrease_allowance(
            deps, env, info, spender, amount, expires,
        )?),
        ExecuteMsg::TransferFrom {
            owner,
            recipient,
            amount,
        } => Ok(execute_transfer_from(
            deps, env, info, owner, recipient, amount,
        )?),
        ExecuteMsg::SendFrom {
            owner,
            contract,
            amount,
            msg,
        } => Ok(execute_send_from(
            deps, env, info, owner, contract, amount, msg,
        )?),
        ExecuteMsg::UpdateMarketing {
            project,
            description,
            marketing,
        } => Ok(execute_update_marketing(
            deps,
            env,
            info,
            project,
            description,
            marketing,
        )?),
        ExecuteMsg::UploadLogo(logo) => Ok(execute_upload_logo(deps, env, info, logo)?),
    }
}

pub fn update_minter(
    deps: DepsMut,
    info: MessageInfo,
    address: String,
) -> Result<Response, ContractError> {
    let config = TOKEN_INFO.load(deps.storage)?;

    if let Some(minter_data) = config.mint.as_ref() {
        if minter_data.minter != info.sender {
            return Err(ContractError::Unauthorized {});
        }
    };

    let new_minter = deps.api.addr_validate(&address)?;

    TOKEN_INFO.update(deps.storage, |mut state| -> Result<_, ContractError> {
        state.mint = Some(MinterData {
            minter: new_minter,
            cap: None,
        });
        Ok(state)
    })?;
    Ok(Response::new().add_attribute("method", "update_minter"))
}

#[cfg_attr(not(feature = "library"), entry_point)]
pub fn query(deps: Deps, _env: Env, msg: QueryMsg) -> StdResult<Binary> {
    match msg {
        // inherited from cw20-base
        QueryMsg::TokenInfo {} => to_binary(&query_token_info(deps)?),
        QueryMsg::Balance { address } => to_binary(&query_balance(deps, address)?),
        QueryMsg::Allowance { owner, spender } => {
            to_binary(&query_allowance(deps, owner, spender)?)
        }
    }
}

#[cfg_attr(not(feature = "library"), entry_point)]
pub fn migrate(_deps: DepsMut, _env: Env, _msg: MigrateMsg) -> StdResult<Response> {
    Ok(Response::default())
}
```

As the code is using cw20-base library on almost all methods the tests from this repo will cover only minting and change owner features.

> /tokens-factory/contracts/token/src/test.rs
```Rust
#[cfg(test)]
mod test {
    #[cfg(test)]
    use crate::contract::instantiate;
    use crate::{contract::execute, msg::ExecuteMsg, ContractError};
    use cosmwasm_std::{
        testing::{mock_dependencies, mock_env, mock_info},
        DepsMut, Uint128,
    };
    use cw20::{Cw20Coin, MinterResponse, TokenInfoResponse};
    use cw20_base::{
        contract::{query_balance, query_token_info},
        msg::InstantiateMsg,
    };

    /* Generic method to instantiate the contract without repeating
    this code many times */
    fn do_instantiate(mut deps: DepsMut) -> TokenInfoResponse {
        // GIVEN
        let instantiate_msg = InstantiateMsg {
            name: "Bit Money".to_string(),
            symbol: "BTM".to_string(),
            decimals: 2,
            mint: Some(MinterResponse {
                minter: "creator".to_string(),
                cap: Some(Uint128::new(1234)),
            }),
            initial_balances: vec![Cw20Coin {
                amount: Uint128::new(123),
                address: "terra1x46rqay4d3cssq8gxxvqz8xt6nwlz4td20k38v".to_string(),
            }],
            marketing: None,
        };
        let info = mock_info("creator", &[]);
        let env = mock_env();

        // WHEN
        let res = instantiate(deps.branch(), env, info, instantiate_msg).unwrap();

        // THEN
        assert_eq!(0, res.messages.len());
        let token_info = query_token_info(deps.as_ref()).unwrap();
        assert_eq!(
            token_info,
            TokenInfoResponse {
                name: "Bit Money".to_string(),
                symbol: "BTM".to_string(),
                decimals: 2,
                total_supply: Uint128::new(123),
            }
        );
        token_info
    }

    #[test]
    fn test_mint_new_token_units() {
        // GIVEN
        let mut deps = mock_dependencies(&[]);
        let info = mock_info("creator", &[]);
        let env = mock_env();
        let msg = ExecuteMsg::Mint {
            recipient: "creator".into(),
            amount: Uint128::new(123),
        };

        // WHEN
        do_instantiate(deps.as_mut());
        let res = execute(deps.as_mut(), env, info, msg).unwrap();

        // THEN
        assert_eq!(0, res.messages.len());
        assert_eq!(
            query_token_info(deps.as_ref()).unwrap(),
            TokenInfoResponse {
                name: "Bit Money".to_string(),
                symbol: "BTM".to_string(),
                decimals: 2,
                total_supply: Uint128::new(246),
            }
        );
        assert_eq!(
            query_balance(deps.as_ref(), "creator".to_string())
                .unwrap()
                .balance,
            Uint128::new(123)
        );
    }

    #[test]
    fn test_update_minter() {
        // GIVEN
        let mut deps = mock_dependencies(&[]);
        let info = mock_info("creator", &[]);
        let new_minter_info = mock_info("terra1x46rqay4d3cssq8gxxvqz8xt6nwlz4td20k38v", &[]);
        let env = mock_env();
        let update_minter_msg = ExecuteMsg::UpdateMinter {
            address: "terra1x46rqay4d3cssq8gxxvqz8xt6nwlz4td20k38v".to_string(),
        };
        let mint_msg = ExecuteMsg::Mint {
            recipient: "terra1x46rqay4d3cssq8gxxvqz8xt6nwlz4td20k38v".into(),
            amount: Uint128::new(123),
        };

        // WHEN
        do_instantiate(deps.as_mut());
        let update_minter_res = execute(deps.as_mut(), env.clone(), info, update_minter_msg).unwrap();
        execute(deps.as_mut(), env, new_minter_info, mint_msg).unwrap();

        // THEN
        assert_eq!(0, update_minter_res.messages.len());
        assert_eq!(
            query_token_info(deps.as_ref()).unwrap(),
            TokenInfoResponse {
                name: "Bit Money".to_string(),
                symbol: "BTM".to_string(),
                decimals: 2,
                total_supply: Uint128::new(246),
            }
        );
        assert_eq!(
            query_balance(
                deps.as_ref(),
                "terra1x46rqay4d3cssq8gxxvqz8xt6nwlz4td20k38v".to_string()
            )
            .unwrap()
            .balance,
            Uint128::new(246)
        );
    }

    #[test]
    fn test_unauthorized_minting() {
        // GIVEN
        let mut deps = mock_dependencies(&[]);
        let info = mock_info("terra1x46rqay4d3cssq8gxxvqz8xt6nwlz4td20k38v", &[]);
        let env = mock_env();
        let mint_msg = ExecuteMsg::Mint {
            recipient: "terra1x46rqay4d3cssq8gxxvqz8xt6nwlz4td20k38v".into(),
            amount: Uint128::new(123),
        };

        // WHEN
        do_instantiate(deps.as_mut());
        let err = execute(deps.as_mut(), env, info, mint_msg).unwrap_err();

        // THEN
        assert_eq!(ContractError::Unauthorized {}, err);
    }
}
```

To complete the smart contract and be able to deploy you must modify schemas that way you can have the models defined to be able to instantiate the contract and interact with the contract from the frontend:

> /tokens-factory/contracts/token/examples/schemas.rs
```Rust
use std::env::current_dir;
use std::fs::create_dir_all;

use cosmwasm_schema::{export_schema, remove_schemas, schema_for};

use cw20_base::msg::InstantiateMsg;
use token::msg::{ExecuteMsg, QueryMsg};

fn main() {
    let mut out_dir = current_dir().unwrap();
    out_dir.push("schema");
    create_dir_all(&out_dir).unwrap();
    remove_schemas(&out_dir).unwrap();

    export_schema(&schema_for!(InstantiateMsg), &out_dir);
    export_schema(&schema_for!(ExecuteMsg), &out_dir);
    export_schema(&schema_for!(QueryMsg), &out_dir);
}
```
Execute `cargo schema` inside `/tokens-factory/contracts/token` folder to generate the new schema. Afterwords you must edit `instantiateMsg` property `terrain.config.json` sending the instantiate configuration:


> /tokens-factory/terrain.config.json
```Json
{
  "_global": {
    "_base": {
      "instantiation": {
        "instantiateMsg": {
          "name": "Bit Money",
          "symbol": "BTM",
          "decimals": 2,
          "initial_balances": [
            {
              "amount": "123",
              "address": "terra1x46rqay4d3cssq8gxxvqz8xt6nwlz4td20k38v"
            }
          ]
        }
      }
    }
  }
  // ...
}
```

:::{tip}
Deploy the contract again to validate that everything works as expected. In any case [here](https://github.com/emidev98/token-factory/commit/f0c6d4979daaf72e30f7b913bb6dcf69100f8b36) you can see the changes done until now. If your code is not working as expected clone the following repo and continue from this point using: 
```
> git clone -n https://github.com/emidev98/tokens-factory
> cd tokens-factory
> git checkout f0c6d4979daaf72e30f7b913bb6dcf69100f8b36
```
:::
