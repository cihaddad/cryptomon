# Cryptomon
Blockchain-based game built on EOSIO platform! This project was built using eosio-cdt ver. 1.6. 

Gameplay:
The objective of cryptomon is to collect rare crypto-monsters that are each randomized to have uniqueness and take care of them. Each cryptomon has their own needs both physically and mentally and it is up to you to make sure they are healthy. If neglected, ignored, or abused, it will result in their death! A cryptomon can live forever but if dead, they cannot be brought back to life.

# Methods (Design & Implementation)
To perform smart-contract development on this platform, the class representing the smart-contract extends the base class contract
present in the platform’s core implementation. To modularize the smart-contract, a header file is created with class, method, user-defined data structures declared and a cpp file with definition and implementation of said entities.

The natural flow of an account on chain interacting with the smart contract is as follows: an account creates their Player uniquely identified by eosio::name type, the Player either purchases a Cryptomon from the marketplace or creates their own Cryptomon, player interacts/cares for their Cryptomon, and lastly, the player has the option to either sell, trade, or retain their Cryptomon. In order to do what was described above, data structures that represent a Player, a Cryptomon, and a Transaction event whether it is buy, sell, or trade must be defined. These data structures are to be stored on-chain for use. In EOSIO, persistent storage of data takes the form of a multi-index table where a unique primary key of type uint64_t is required, and other indexes can be of any type. Three tables are created for each entity: Player, Cryptomon, and Transaction event.

The players table contains the fields: eosio::name key, vector of uint8_t, std::string playerName, eosio::asset funds, and uint64_t cryptomon_index. The field with identifier “key” is the primary key of this table and is unique to each account on the platfom. In order for players to manage their store of value, tokens, for buying/selling/trading, we have the eosio::asset funds field!

The cryptomon table contains the fields: uint64_t key, uint8_t HP, uint8_t level, uint8_t happiness, uint8_t health, uint8_t hunger, eosio::time_point start, eosio::time_point current, and std::string mon_name where “key” is the primary-key of table and is auto-incremented per every entry into table to ensure uniqueness. Note, each player owns their own Cryptomon and to establish this relation, the value of cryptomon_index field in the player table is the key value of the cryptomon table. Each Cryptomon has attributes like HP that are dependent on time elapsed between certain interactions amongst the player and their Cryptomon which is the reason for having the field eosio::time_point current. This field is updated to the last time of interaction and compared with current time to take the difference between interactions to make necessary adjustments.

The transaction event table, transact, contains the fields: eosio::name account_one, eosio::name account_two,
uint64_t cryptomon_index, bool swap, eosio::asset price where the primary key is cryptomon_index. There are two states an entry into this table could be in: 1) a pending trade state, 2) a pending sell state. The difference in the two states is given by the value of the eosio::name account_two field. If eosio::name account_two field is not empty, this denotes a trade event otherwise it is a sell event. In the selling state, price corresponds to the cost of the Cryptomon offered by the player. In the trade state, account_one is the initiator of trade with account_two and the “price” is how much they are offering to account_two in the trade. Swap is either true if initiator is also giving their cryptomon.

# Contract Actions
upsertplayer(): creates entry in players table or updates existing Player's playerName field
createmon(): creates entry in cryptomons for callee (Player) if they have none, and sets cryptomon_index in players table to key of\n Cryptomon
playerfund(): if a transfer to contract's account occurs, deposit asset to respective player
deleteplayer(): deletes player from players table associated with account if it exists
deletemon(): deletes entry in cryptomons table of the Cryptomon that corresponds with the callee (Player)
listmon(): creates entry in transacts table of a sell event of a player's Cryptomon
purchasemon(): purchase Cryptomon that is listed in transacts table if player has no Cryptomon and deletes entry from table if successful
accepttrade(): accept trade created by initiator, deletes entry of trade event from transacts table
setname(): sets name of a player's Cryptomon
interact(): measures difference between a player’s Cryptomon current time point and current time for effect and updates Cryptomon’s current time point to now
itemapply(): apply item in inventory to Cryptomon for effect on its attributes, remove item from inventory
itemdelete(): delete specified item in inventory of player
withdraw(): move asset from contract account to account
canceltrade(): cancels existing trade event in transacts table, removes entry from table
inittrade(): initiate trade to another player specifying offering amount of tokens and swap (optional), creates a trade event entry in transact table
monversion(): returns the version of Cryptomon game
