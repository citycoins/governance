# CCIP-001

## Preamble

| CCIP Number   | 001                                   |
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

[^1]: https://stacking.club
[^2]: https://github.com/stacksgov/sips/blob/main/sips/sip-010/sip-010-fungible-token-standard.md
[^3]: https://docs.stacks.co/understand-stacks/proof-of-transfer

## Specification

After a set of CityCoin contracts are deployed, an additional activation step must occur before any additional functions become available.

A threshold is set within the contract through which anyone can send a transaction as a signal for activating the CityCoin.

Once that threshold is met, the current block height of the final transaction is added to the activation delay set in the contract, calculating the block height at which mining and stacking become available.

There are no CityCoins minted or distributed prior to the start of mining.

## Related Work

## Backwards Compatibility

None, intial implementation-style.

## Activation

## Reference Implementations

MiamiCoin (MIA)

NewYorkCityCoin (NYC)
