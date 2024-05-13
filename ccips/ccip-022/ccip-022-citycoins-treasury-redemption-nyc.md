# CCIP-022

## Preamble

| CCIP Number   | 022                                                  |
| ------------- | ---------------------------------------------------- |
| Title         | CityCoins Treasury Redemption NYC                    |
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

Following that change this CCIP proposes a redemption mechanism for NYC token holders to be fairly compensated by redeeming their CityCoins for a portion of the STX held in the city treasury.

## Specification

### Redemption Extensions

This CCIP will implement a new redemption extension for NYC.

The redemption extensions will:

- receive and hold funds from the respective city treasury contract
- record the total supply of NYC at the time of initialization
- calculate the ratio of NYC to STX based on the total supply and contract balance
- allow users to burn NYC in exchange for their portion of STX and track the related totals

## Backwards Compatibility

This CCIP is supplemental to CCIP-020[^1] and not backwards compatible.

## Activation

This CCIP will be voted on using a vote contract that adheres to CCIP-015[^2] using the last two active stacking cycles for the protocol.

- MIA cycles 82 and 83
- NYC cycles 82 and 83

No scale factor is required.

Voting will begin when the contract is deployed and continue until the proposal is executed.

This CCIP follows the same voting criteria as CCIP-020[^1] except only NYC votes are counted.

- the votes are tallied and available in read-only functions
- the proposal will not pass unless 25% of NYC have participated

## Reference Implementations

- ccip022-citycoins-treasury-redemption-nyc.clar[^3]
- ccd012-redemption-nyc.clar[^4]

## Footnotes

[^1]: https://github.com/citycoins/governance/blob/main/ccips/ccip-020/ccip-020-graceful-protocol-shutdown.md
[^2]: https://github.com/citycoins/governance/blob/main/ccips/ccip-015/ccip-015-community-proposal-voting-process.md
[^3]: TBD
[^4]: TBD
