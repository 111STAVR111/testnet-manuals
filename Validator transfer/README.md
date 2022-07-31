# Validator transfer using the Stride node for the Cosmos ecosystem as an example
Validators take more responsibility by raising a node, because they are responsible for the safety of delegates' funds in addition to their own steak. Validators cannot take direct possession of delegated funds, but their mistakes (when slashing) can cause the loss of some of the delegates' and their own funds. This can happen for two reasons:

### > Poor operation of the validator
When the validator node stops processing blocks and misses enough blocks. In this case, the validator is in jail, but can get out of jail. The amount of jailed funds is reduced by 0.01%.

### > Double signature
When the validator signs two identical blocks at the same height. In this case, the validator is imprisoned and can't get out of it. The share of the jailed funds is reduced by 5 percent. Double signing can happen if one `priv_validator_key.json` file is used on different servers, which is categorically not allowed.
