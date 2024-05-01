# CCIP-022

## Preamble

| CCIP Number   | 022                                                  |
| ------------- | ---------------------------------------------------- |
| Title         | CityCoins Treasury Redemption                        |
| Author(s)     | Raphael R. Sierra rapha@fontainebleau-management.com |
|               | Jason Schrader jason@joinfreehold.com                |
| Consideration | Governance                                           |
| Type          | Standard                                             |
| Status        | Draft                                                |
| Created       | 2024-04-23                                           |
| License       | BSD-2-Clause                                         |
| Supplements   | CCIP-020                                             |

## Introduction

With the execution of CCIP-020 Graceful Protocol Shutdown[^1] the functions for mining and stacking CityCoins are disabled.

Following that change this CCIP proposes a redemption mechanism for MIA and NYC token holders to be fairly compensated by redeeming their CityCoins for a portion of the STX held in the city treasuries.

## Specification

### Redemption Extensions

This CCIP will implement two new redemption extensions, one each for MIA and NYC.

The redemption extensions will:

- receive and hold funds from the respective city treasury contract
- record the total supply of MIA/NYC at the shutdown block height
- calculate the value of the STX pair based on the total supply and contract balance
- allow users to convert their CityCoins balance to STX and track the related totals

## Backwards Compatibility

This CCIP is supplemental to CCIP-020[^1] and not backwards compatible.

## Activation

This CCIP will be voted on using a vote contract that adheres to CCIP-015[^2] using the same two stacking cycles as CCIP-020[^1] and CCIP-021[^3].

- MIA cycles 80 and 81
- NYC cycles 80 and 81

The scale factor for MIA was determined using the same formula used in CCIP-015 and calculated based on the total supply at the start block of the first cycle and the end block of the last cycle.

- MIA scale factor: 0.8916 _(prev: 0.8823)_

The calculations used for the scale factor are available in the supplemental spreadsheet[^4].

Voting will begin when the contract is deployed and continue until the proposal is executed.

This CCIP also follows the same voting criteria as CCIP-020[^1], such that:

- the votes are tallied per city and available in read-only functions
- the proposal will not pass unless 25% of MIA or NYC have participated

## Reference Implementations

- ccip-022-citycoins-treasury-redemption[^5]

## Footnotes

[^1]: https://github.com/citycoins/governance/blob/main/ccips/ccip-020/ccip-020-graceful-protocol-shutdown.md
[^2]: https://github.com/citycoins/governance/blob/main/ccips/ccip-015/ccip-015-community-proposal-voting-process.md
[^3]: https://github.com/citycoins/governance/blob/feat/add-ccip-021/ccips/ccip-021/ccip-021-extend-direct-execute-sunset-period-2.md
[^4]: See the [ccip-022-vote-calculations-per-ccip-015 spreadsheet](./ccip-022-vote-calculations-per-ccip-015.ods)
[^5]: https://github.com/citycoins/governance/blob/feat/add-ccip-022/ccips/ccip-022/ccip-022-citycoins-treasury-redemption.md
