# CCIP-011

## Preamble

| CCIP Number   | 011                                   |
| ------------- | ------------------------------------- |
| Title         | CityCoins Stacked Tokens Voting       |
| Author(s)     | Jason Schrader jason@joinfreehold.com |
| Consideration | Technical, Governance                 |
| Type          | Standard                              |
| Status        | Draft                                 |
| Created       | 2022-03-25                            |
| License       | BSD-2-Clause                          |
| Replaces      | CCIP-006                              |

## Introduction

The Stacks blockchain is a Layer 1 blockchain connected to Bitcoin, in which miners spend Bitcoin to bid for and win a fixed amount of Stacks tokens. Stackers have the option to lock up Stacks tokens for a specified amount of time, and in turn, receive a portion of the Bitcoin spent by miners proportionate to the amount Stacked[^1].

Taking this concept a level further, a new token (a "CityCoin") is created on the Stacks blockchain, following the SIP-010 fungible token standard[^2], such that CityCoins can be mined and Stacked per the methods stated above except a portion of the miner's bid would be redirected to the city's wallet overseen by a trusted third party custodian.

CityCoins leverage similar properties from the Proof of Transfer (PoX) consensus mechanism[^3] of the Stacks blockchain, programmed through a smart contract in Clarity.

This proposal describes the on-chain vote method used to ratify CCIP-008, CCIP-009, and CCIP-010.

## Specification

The voting contract is designed similarly to the SIP-012 voting process[^4], in which the user's vote is equal to the amount of tokens stacked across a defined set of reward cycles.

Through the `vote` function, a principal can cast their vote for a defined proposal and the contract will:

### Proposal Data

TODO: define proposal data

### Vote Data

TODO: define vote data

### Vote Parameters

TODO: define vote parameters

### Vote Calculations

TODO: define vote calculations

## Backwards Compatibility

None, this is the initial implementation.

## Activation

None, this is the initial implementation.

## Reference Implementations

TODO: add references

## Footnotes

[^1]: https://stacking.club
[^2]: https://github.com/stacksgov/sips/blob/main/sips/sip-010/sip-010-fungible-token-standard.md
[^3]: https://docs.stacks.co/understand-stacks/proof-of-transfer
[^4]: TODO: SIP-012 vote link
