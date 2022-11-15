# Code Journal 1

In this code journal I will be going over the State.rs file where the logic that allows the contract to presist attributing the data a unique key that allows it to be indexed.

 ---

### Imports From Dependencies 

Starting at the top of our file we have the basic import from the dependencies that allow us to create the logic for the State.

```
use schemars::JsonSchema;
use serde::{Deserialize, Serialize};
```
Above we have the first two imports, the first from schemars - we import JsonSchema, a declarative language that allows us to anototate and validate the data in Json.

The second from serde that will handle the serialization and deserialization of the said data.

Next we also import from cosmwasm standard library an address which under the hood is a simple string.

And finally we use a helpes provided by storage plus that allows us to store items in storage and retrieve them with Map. 
```
use cosmwasm_std::Addr;
use cw_storage_plus::{Item, Map};
```
---
### Basic Template Struct

When we generate the basic smart contract template, the state.rs comes with a single struct:

```
#[derive(Serialize, Deserialize, Clone, Debug, PartialEq, Eq, JsonSchema)]
pub struct State {
    pub count: i32,
    pub owner: Addr,
}
```
This is comprised of the macro that describes the implementations being used.

```
#[derive(Serialize, Deserialize, Clone, Debug, PartialEq, Eq, JsonSchema)]
```
And the actual State Struct that contains an i32 interger and a the address that is a string

```
pub struct State {
    pub count: i32,
    pub owner: Addr,
}
```
---
### Storage

At the very end of the template a const is declared to allow the data to be stored by giving it a specific storage key which also is used to retrieve the data. In this case the storage key is "state"

```
pub const STATE: Item<State> = Item::new("state");
```
---

### Beyond the Template

The file state.rs contais the basic code above when the template is generated and it will work with whatver code we add as long as the logic throughout the rest of the contract keeps true to it. 
However, upon studying other contracts, depending on the needs of the smart contract, the state.rs will change substantially. 

In this example, the smart contract was created to allow voting. 

### Imports From Dependencies 

Similarities:
They still use the cw_storage_plus dependency to store and retrive data. They also use the cosmwasm standar library to import the adress

Differences:

They don't use the JsonSchema, instead they use a cosmwasm schema library to import cw_serde. And they add an interger along with the address import

```
use cosmwasm_schema::cw_serde;
use cosmwasm_std::{Addr, Uint128};
use cw_storage_plus::{Item, Map};
```

### State Struct

The macro now is only the cw_serde.
They still declare the owner as the specific address being imported. But thet add the interger they also inported as the staked tokens. 

the other two implementations are the poll_count which is a 56 bit unsigned interger type, and the denom which is type string.

```
#[cw_serde]
pub struct State {
    pub denom: String,
    pub owner: Addr,
    pub poll_count: u64,
    pub staked_tokens: Uint128,
}
```
### Other Structs

In this example, they implement 3 more structs:
- Token Manager : to track the total stake balance, the number of tokens depositd to vote through the poll_id and the actual poll_id of the ones who paticipated.
```
#[cw_serde]
#[derive(Default)]
pub struct TokenManager {
    pub token_balance: Uint128,             
    pub locked_tokens: Vec<(u64, Uint128)>,
    pub participated_polls: Vec<u64>,      
}
```

+ Voter: This struct tracks the vote and the number of tokes deposit. 

```
#[cw_serde]
pub struct Voter {
    pub vote: String,
    pub weight: Uint128,
}
```
- Poll: All the information about the poll; creator, the status, type of vote (yes,no), the voter info, the description, etc.

```
#[cw_serde]
pub struct Poll {
    pub creator: Addr,
    pub status: PollStatus,
    pub quorum_percentage: Option<u8>,
    pub yes_votes: Uint128,
    pub no_votes: Uint128,
    pub voters: Vec<Addr>,
    pub voter_info: Vec<Voter>,
    pub end_height: u64,
    pub start_height: Option<u64>,
    pub description: String,
}
```

### Enum

Thet also created an enum that allows to track the status of a prticular poll

```
#[cw_serde]
pub enum PollStatus {
    InProgress,
    Tally,
    Passed,
    Rejected,
}
```

### Storage and Retriving Data

Finally at the end of the file there are our consts that allow to store the Data and to retrieve it.

In this case we have the basic storage const like we see in the generate template. Only in this case was givena different name and storage key - Config

```
pub const CONFIG: Item<State> = Item::new("config");
```
The other two use the Map implementation with two values (unsigned interger, Poll and TokenManager value). And a key (in this case "polls" and "bank") that will be generated as an  UUID clientside.

```
pub const POLLS: Map<&[u8], Poll> = Map::new("polls");
pub const BANK: Map<&[u8], TokenManager> = Map::new("bank");
```

### Final Thoughts

Although this a simple smart contract that allows voting through token deposit, it helped understand better how the state.rs file can be expanded to store specific data to the smart contract being built. 
