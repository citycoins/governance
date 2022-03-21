# CCIP-006

## Preamble

| CCIP Number   | 006                                   |
| ------------- | ------------------------------------- |
| Title         | CityCoins VRF                         |
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

This proposal describes the verifiable random function (VRF) used to calculate the block winner for a CityCoin.

## Specification

The VRF contract is used to determine the winner of a mined block.

Through the `get-random-uint-at-block` function, the contract:

- fetches the on-chain VRF proof from the block height
- converts the lower 16 bytes into a uint
- returns the random integer

More information on Stacks block assembly can be found in [SIP-005](https://github.com/stacksgov/sips/blob/main/sips/sip-005/sip-005-blocks-and-transactions.md#blocks).

## Backwards Compatibility

None, as this proposal is the initial implementation.

## Activation

None, as this proposal is the initial implementation.

## Reference Implementations

- `citycoin-vrf` deployed on the Stacks mainnet[^4], implementing the `get-random-uint-at-block` function on lines 05-15

## Footnotes

[^1]: https://stacking.club
[^2]: https://github.com/stacksgov/sips/blob/main/sips/sip-010/sip-010-fungible-token-standard.md
[^3]: https://docs.stacks.co/understand-stacks/proof-of-transfer
[^4]: https://explorer.stacks.co/txid/SP466FNC0P7JWTNM2R9T199QRZN1MYEDTAR0KP27.citycoin-vrf?chain=mainnet
