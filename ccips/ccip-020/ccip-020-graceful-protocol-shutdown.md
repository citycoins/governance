# CCIP-020

## Preamble

| CCIP Number   | 014                                   |
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

As the CityCoins Protocol prepares for the upcoming Nakamoto release on the Stacks blockchain, the accumulation of technical debt with each change has made it increasingly challenging for core contributors and volunteers to keep up with the necessary updates.

In light of these challenges and after careful consideration of community feedback, this CCIP proposes a graceful shutdown of the CityCoins Protocol. The shutdown process will involve disabling the mining and stacking functionality while ensuring that MIA and NYC token holders are fairly compensated through a redemption mechanism utilizing the STX held in the city treasuries.

The primary objectives of this CCIP are to:

- Protect the interests of MIA and NYC token holders
- Honor the milestones achieved by the CityCoins Protocol
- Honor the gift agreement with the City of Miami
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

For example, at the time of writing this we are in cycle 80. If a user is stacking from cycles 77-84 then the following would be true:

| Cycle | Claim STX | Claim CityCoins | Description               |
| ----- | --------- | --------------- | ------------------------- |
| 77    | Yes       | No              | same as before            |
| 78    | Yes       | No              | same as before            |
| 79    | Yes       | No              | same as before            |
| 80    | Yes       | No              | same as before            |
| 81    | No        | No              | shutdown, no STX to claim |
| 82    | No        | No              | shutdown, no STX to claim |
| 83    | No        | No              | shutdown, no STX to claim |
| 84    | No        | Yes             | retrieve locked CityCoins |

### Redemption Extensions

This CCIP will implement two new redemption extensions, one each for MIA and NYC.

The redemption extensions will:

- receive and hold funds from the respective city treasury contract
- capture the total supply of MIA/NYC at the shutdown block height
- calculate the value of the STX pair based on the total supply and balance
- allow users to convert their CityCoins balance to STX (similar to V1->V2)

## Backwards Compatibility

This CCIP is not backwards compatible.

## Activation

This CCIP will be voted on using a vote contract that adheres to CCIP-015 using the last two active cycles from when the contract is deployed.

Currently, this would be:

- MIA cycles 79 and 80
- NYC cycles 79 and 80

The scale factor for MIA was determined using the same formula used in CCIP-015 and calculated based on the total supply at the start block of the first cycle and the end block of the last cycle.

- MIA scale factor: TBD

The calculations used for the scale factor are available in the supplemental spreadsheet.

## Reference Implementations

TBD

## Footnotes

TBD
