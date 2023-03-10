# **Lucky Roll**

Lucky Roll is a lucky game that uses fair random number from [Nois](https://docs.nois.network/) - a Proof of Stake blockchain protocol that allows developers to use secure, unbiased and cost efficient randomness via IBC

The game is hosted on [Aura Network](https://aura.network/)

## Game Flow

### 1. Game Owner:
* Owner `Instantiate` contract with following parameters
    - `nois_proxy` is nois proxy contract address in our chain
    - `time_start` is time start of game, format "2023-01-19 19:05:00Z" UTC+0
    - `time_end` is time stop of game, format "2023-01-19 19:05:00Z" UTC+0
* Owner can execute `reset` command to reset game with same parameters as `Instantiate`
* Owner execute `set_white_list` command to specify who can allowed to playing game
* Onwer execute `set_prizes` command to set prizes
* The game will be played in rounds, every time the `roll` command is successful, the game will end a round. To play again, use the command `reset`

* In the end, owner execute `roll` command to distribute prizes for players

### 2. Participants
* Players execute `lucky_number` command to receive owned lucky number
example:
```
    export LUCKY_ROLL_CONTRACT=aura1r5xw9m0tlwlqg3uhseqq27tzhsn7m5vlc32fen977hswa8f2gy5shfjt43
    
    aurad tx wasm execute $LUCKY_ROLL_CONTRACT \
       '{"lucky_number" :{}}' \
       --from <your-validator-signing-key> \
       --chain-id euphoria-2 \
       --gas-prices 0.025ueaura \
       --gas=auto \
       --gas-adjustment 1.4 \
       --node=https://rpc.euphoria.aura.network:443/ \
       --broadcast-mode=block \
       --amount 300ueaura
```

* You can also query the state of all attendees using:
```
aurad query wasm contract-state smart \
        $LUCKY_ROLL_CONTRACT  \
        '{"get_attendees": {}}' \
        --node=https://rpc.euphoria.aura.network:443/
```

The game will allow a period from `time_start` to `time_end` to wait for all players to roll. After this period, the owner can execute command `roll` to distribute the prize

# Sample commands
`reset`
```Rust
    Reset {
        nois_proxy: String,
        time_start: String,
        time_end: String,
    }
```
- only allow onwer to execute
- will reset game

`set_white_list`
```Rust
    SetWhiteList {
        attendees: Vec<String>
    }
```
- only allow owner to execute
- only executed in round
- specific players by wallet address

`set_prizes`
```Rust
    Setprizes {
        prizes: Vec<String>
    }
```
- only allow owner to execute
- only executed in round
- this will make a message to Nois Proxy and will be shuffled randomly using NoisCallback (after execute `set_prizes` command, need to wait 1-2' for **NoisCallback** then you can execute another command like `roll`)

`roll`
```Rust
    Roll {

    }
```
- only allow owner to execute
- only executed in round
- only executed before `time_end`
- this command will distribute prizes for all attendees base on prizes list and attendee's lucky number
- if success, it will end a round

`lucky_number`
```Rust
    LuckyNumber {

    }
```
- only allow WhiteList member to execute
- only executed before `time_start` and after `time_end`
- only executed in round
- make a message to Nois Proxy and receive NoisCallback as lucky number

# Query
`get_prizes`
```Rust
    GetPrizes{}
```
- list all prizes

`get_attendees`
```Rust
    GetAttendees{}
```
- list all attendees

`get_distribute_prizes`
```Rust
    GetDistributePrizes{}
```
- list attendees and their prize

# Command

* `build wasm file`
```
    beaker wasm build
```

* `Store wasm file`
```
    export CODE_ID=$(aurad tx wasm store \
       target/wasm32-unknown-unknown/release/lucky_roll.wasm \
       --from <your-key> \
       --chain-id euphoria-2 \
       --gas=auto \
       --gas-adjustment 1.4  \
       --gas-prices 0.025ueaura \
       --broadcast-mode=block \
       --node=https://rpc.euphoria.aura.network:443/ -y \
       |yq -r ".logs[0].events[1].attributes[1].value" ) 
```

* `Instantiate contract`
```
    export NOIS_PROXY=aura19z2hv8l87qwg8nnq6v76efjm2rm78rkdghq4rkxfgqrzv3usw8lq26rmwt
    export TIME_START="2023-01-21 00:00:00Z"
    export TIME_END="2023-01-21 23:59:59Z"
    export LUCKY_ROLL_CONTRACT=$(aurad \
       tx wasm instantiate $CODE_ID \
       '{"nois_proxy": "'"$NOIS_PROXY"'","time_start":"'"$TIME_START"'","time_end":"'"$TIME_END"'"}' \
       --label=lucky-roll \
       --no-admin \
       --from <your-key> \
       --chain-id euphoria-2 \
       --gas=auto \
       --gas-adjustment 1.4 \
       --broadcast-mode=block \
       --node=https://rpc.euphoria.aura.network:443/ -y \
       |yq -r '.logs[0].events[0].attributes[0].value' )
```

* `Execute`
example
```
    aurad tx wasm execute $LUCKY_ROLL_CONTRACT \
        '{"lucky_number": {}}' \
        --from <your-key> \
        --gas-prices 0.025ueaura \
        --gas=auto \
        --gas-adjustment 1.3 \
        --node=https://rpc.euphoria.aura.network:443/ \
        --amount 300ueaura
```

* `Query`
example
```
    aurad query wasm contract-state smart \
        $LUCKY_ROLL_CONTRACT  \
        '{"get_attendees": {}}' \
        --node=https://rpc.euphoria.aura.network:443/
```
