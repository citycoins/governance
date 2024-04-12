# CCIP-020

## Preamble

| CCIP Number   | 020                                   |
| ------------- | ------------------------------------- |
| Title         | Graceful Protocol Shutdown            |
| Author(s)     | Jason Schrader jason@joinfreehold.com |
| Consideration | Governance, Economic                  |
| Type          | Standard                              |
| Status        | Draft                                 |
| Created       | 2024-03-22                            |
| License       | BSD-2-Clause                          |
| Supplements   | CCIP-012, CCIP-013, CCIP-017          |

## Introduction

As the CityCoins Protocol prepares for the upcoming Nakamoto release on the Stacks blockchain, the accumulation of technical debt with each change has made it increasingly challenging for core contributors and volunteers to keep up with the necessary updates. In light of these challenges and after careful consideration of community feedback, this CCIP proposes a graceful shutdown of the CityCoins Protocol.

The shutdown process will involve disabling the mining and stacking functionality, and the primary objectives of this CCIP are to:

- Protect the interests of MIA and NYC token holders
- Honor the milestones achieved by the CityCoins Protocol
- Facilitate a smooth transition and minimize disruption to the ecosystem

## Specification

### Disable Mining

This CCIP will disable mining in the current mining contract, such that:

- Mining will stop at the block height the shutdown occurs
- Any mining transactions sent after the shutdown block height will fail
- Any mining claims made in this and prior contracts will still function

Mining claims are based on block heights and should be routed to the correct contracts:

| Contract                  | Start Block | End Block |
| ------------------------- | ----------- | --------- |
| miamicoin-core-v1         | 24497       | 58917     |
| newyorkcitycoin-core-v1   | 37449       | 58922     |
| miamicoin-core-v2         | 58921       | 96779     |
| newyorkcitycoin-core-v2   | 58925       | 96779     |
| ccd006-citycoin-mining    | 96779       | 107389    |
| ccd006-citycoin-mining-v2 | 107389      | TBD       |

### Disable Stacking

This CCIP will disable stacking in the current stacking contract, such that:

- Stacking will stop at the block height the shutdown occurs
- Any stacking transactions sent after the shutdown block height will fail
- Any stacking claims made in this and prior contracts will still function
- Any locked CityCoins can be claimed from the future cycle(s)

For example, at the time of writing this we are in cycle 82. If a user is stacking from cycles 80-86 then the following would be true:

| Cycle | Claim STX | Claim CityCoins | Description               |
| ----- | --------- | --------------- | ------------------------- |
| 80    | Yes       | No              | same as before            |
| 81    | Yes       | No              | same as before            |
| 82    | Yes       | No              | same as before            |
| 83    | No        | No              | shutdown, no STX to claim |
| 84    | No        | No              | shutdown, no STX to claim |
| 85    | No        | No              | shutdown, no STX to claim |
| 86    | No        | Yes             | retrieve locked CityCoins |

Stacking claims are based on cycle numbers and should be routed to the correct contracts:

| Contract                 | Start Cycle | End Cycle |
| ------------------------ | ----------- | --------- |
| miamicoin-core-v1        | 1           | 16        |
| newyorkcitycoin-core-v1  | 1           | 10        |
| miamicoin-core-v2        | 17          | 34        |
| newyorkcitycoin-core-v2  | 11          | 28        |
| ccd007-citycoin-stacking | 54          | TBD       |

> [!NOTE]
> Stacking cycles are independent per city until ccd007-citycoin-stacking was implemented, which combined the cycles for both cities and matches the current Stacks stacking cycle numbers.

## Backwards Compatibility

This CCIP is not backwards compatible.

## Activation

This CCIP will be voted on using a vote contract that adheres to CCIP-015 using the last two active cycles from when the contract is deployed.

Currently, this would be:

- MIA cycles 80 and 81
- NYC cycles 80 and 81

The scale factor for MIA was determined using the same formula used in CCIP-015 and calculated based on the total supply at the start block of the first cycle and the end block of the last cycle.

- MIA scale factor: 0.8916 (prev: 0.8823)

The calculations used for the scale factor are available in the supplemental spreadsheet.

## Reference Implementations

TBD

## Footnotes

TBD
