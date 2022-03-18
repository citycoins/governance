# CCIP-002

## Preamble

| CCIP Number   | 002                                   |
| ------------- | ------------------------------------- |
| Title         | CityCoins Activation                  |
| Author(s)     | Jason Schrader jason@joinfreehold.com |
| Consideration | Technical                             |
| Type          | Standard                              |
| Status        | Draft                                 |
| Created       | 2022-03-16                            |
| License       | BSD-2-Clause                          |

## Introduction

The Stacks blockchain is a Layer 1 blockchain connected to Bitcoin, in which miners spend Bitcoin to bid for and win a fixed amount of Stacks tokens. Stackers have the option to lock up Stacks tokens for a specified amount of time, and in turn, receive a portion of the Bitcoin spent by miners proportionate to the amount Stacked[^1].

Taking this concept a level further, a new token (a "CityCoin") is created on the Stacks blockchain, following the SIP-010 fungible token standard[^2], such that CityCoins can be mined and Stacked per the methods stated above except a portion of the miner's bid would be redirected to the city's wallet overseen by a trusted third party custodian.

CityCoins leverage similar properties from the Proof of Transfer (PoX) consensus mechanism[^3] of the Stacks blockchain, programmed through a smart contract in Clarity.

This proposal describes the registration and activation process for a CityCoin.

## Specification

After a set of CityCoin contracts are deployed, an additional activation step is required before the functions for mining and stacking become available.

There are no CityCoins minted or distributed prior to the start of mining.

Within the core contract are variables to track the activation status, block height, delay, threshold, and registered users count.

Through the `register-user` function, a principal can register their Stacks address as a user until the threshold is reached. This allows for a the required threshold of independent wallets to signal that the CityCoin should be activated.

Once the activation threshold is reached, the core contract:

- records the current block height of the final registration transaction, and adds it to the activation delay set in the contract
- the calculated block height is the first block at which mining and stacking will become available in the core contract
- the calculated block height is sent to the auth contract via `activate-core-contract` to set the core contract in an active state and record the activation block height
- the calculated block height is sent to the token contract via `activate-token` to set the token as activated and calculate the coinbase thresholds based on the emissions schedule

After the activation threshold is reached, the `register-user` function no longer serves a purpose and fails with `err ERR_ACTIVATION_THRESHOLD_REACHED` when called.

## Backwards Compatibility

None, as this proposal is the initial implementation.

## Activation

None, as this proposal is the initial implementation.

## Reference Implementations

MiamiCoin Contracts:

- `miamicoin-core-v1` deployed on the Stacks mainnet, implementing the `register-user` function on lines 128-167[^4]

NewYorkCityCoin Contracts:

- `newyorkcitycoin-core-v1` deployed on the Stacks mainnet, implementing the `register-user` function on lines 129-168[^5]

## Footnotes

[^1]: https://stacking.club
[^2]: https://github.com/stacksgov/sips/blob/main/sips/sip-010/sip-010-fungible-token-standard.md
[^3]: https://docs.stacks.co/understand-stacks/proof-of-transfer
[^4]: https://explorer.stacks.co/txid/SP466FNC0P7JWTNM2R9T199QRZN1MYEDTAR0KP27.miamicoin-core-v1?chain=mainnet
[^5]: https://explorer.stacks.co/txid/SP2H8PY27SEZ03MWRKS5XABZYQN17ETGQS3527SA5.newyorkcitycoin-core-v1?chain=mainnet
