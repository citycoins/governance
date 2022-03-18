# CCIP-001

## Preamble

| CCIP Number   | 001                                   |
| ------------- | ------------------------------------- |
| Title         | CityCoins Traits                      |
| Author(s)     | Jason Schrader jason@joinfreehold.com |
| Consideration | Technical                             |
| Type          | Standard                              |
| Status        | Draft                                 |
| Created       | 2022-03-16                            |
| License       | BSD-2-Clause                          |

## Introduction

The Stacks blockchain is a Layer 1 blockchain connected to Bitcoin, in which miners spend Bitcoin to bid for and win a fixed amount of Stacks tokens. Stackers have the option to lock up Stacks tokens for a specified amount of time, and in turn, receive a portion of the Bitcoin spent by miners proportionate to the amount Stacked in a cycle of 2,100 Bitcoin blocks.[^1]

Taking this concept a level further, a new token (a "CityCoin") is created on the Stacks blockchain, following the SIP-010 fungible token standard[^2], such that CityCoins can be mined and Stacked per the methods stated above except a portion of the miner's bid would be redirected to the city's wallet overseen by a trusted third party custodian.

CityCoins leverage similar properties from the Proof of Transfer (PoX) consensus mechanism[^3] of the Stacks blockchain, programmed through a smart contract in Clarity.

This proposal describes the Clarity trait definitions for a CityCoin.

## Specification

The trait system in Clarity is a mechanism that allows for simple definition and configuration of a common interface.[^4] Traits define a set of named functions, argument types, and the return type for a contract to be considered an implementation of the trait.

The CityCoins Protocol includes two custom trait definitions.

### Core Contract Trait

The `citycoin-core` trait defines the core functionality of a CityCoin, including the following functions:

- `register-user`: Allows a principal to register for the activation of a deployed CityCoin until the specified threshold is met. _(see CCIP-002)_
- `mine-tokens`: Allows a principal to mine CityCoins at the current block height, and records the related mining data in the core contract. _(see CCIP-003)_
- `claim-mining-reward`: Allows the winning principal to claim the mining reward for a CityCoin at the specified block height. _(see CCIP-003)_
- `stack-tokens`: Allows a principal to stack CityCoins for a specified number of reward cycles. _(see CCIP-004)_
- `claim-stacking-reward`: Allows a principal to claim eligible stacking rewards from a past stacking cycle. _(see CCIP-004)_
- `set-city-wallet`: Allows the auth contract to set the city wallet variable in the core contract _(see CCIP-007)_
- `shutdown-contract`: Allows the auth contract to shutdown the core contract, making mining unavailable and unlocking all stacked tokens. _(see CCIP-007)_

### Token Contract Trait

The `citycoin-token` trait defines the token functionality of a CityCoin, including the following functions:

- `activate-token`: Allows a core contract to set an activation block height and coinbase thresholds for the emissions schedule. _(see CCIP-002)_
- `set-token-uri`: Allows the auth contract to set the token URI variable for a CityCoin. _(see CCIP-007)_
- `mint`: Allows a core contract to mint new CityCoins.
- `burn`: Allows a user to burn CityCoins.[^5]

Note: All CityCoin token contracts also follow the SIP-010] trait specification[^2].

[^1]: https://stacking.club
[^2]: https://github.com/stacksgov/sips/blob/main/sips/sip-010/sip-010-fungible-token-standard.md
[^3]: https://docs.stacks.co/understand-stacks/proof-of-transfer
[^4]: https://www.hiro.so/blog/five-useful-clarity-keywords-functions
[^5]: In the initial `v1.0.0` implementation for MiamiCoin, the `burn` function was only allowed to be called by a core contract, however this was updated in `v1.0.1` released starting with NewYorkCityCoin.
