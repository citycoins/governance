# CCIP-022

## Preamble

| CCIP Number   | 022                                                  |
| ------------- | ---------------------------------------------------- |
| Title         | CityCoins Treasury Redemption NYC                    |
| Author(s)     | Raphael R. Sierra rapha@fontainebleau-management.com |
|               | Jason Schrader jason@joinfreehold.com                |
| Consideration | Governance                                           |
| Type          | Standard                                             |
| Status        | Ratified                                             |
| Created       | 2024-04-23                                           |
| License       | BSD-2-Clause                                         |
| Supplements   | CCIP-020                                             |

## Introduction

With the execution of CCIP-020 Graceful Protocol Shutdown[^1] the functions for mining and stacking CityCoins are disabled.

Following that, this CCIP proposes a redemption mechanism for NYC token holders to be fairly compensated by redeeming their NYC for a portion of the STX held in the NYC mining treasury[^2].

## Specification

### Redemption Extension

This CCIP will implement a new redemption extension for NYC.

The redemption extension will:

- receive and hold funds from the NYC mining treasury contract[^2]
- record the combined total supply of NYC at the time of initialization
- calculate the ratio of NYC to STX based on the total supply and contract balance
- allow users to burn NYC in exchange for their portion of STX
- track and return the redemption information for the contract and its users

### Redemption Mechanism

The NYC mining treasury[^2] currently holds 15.5M STX and upon successful execution of this proposal the entire balance will be transferred to the redemption extension.

Following the transfer and upon initialization from the proposal, the redemption extension will:

- get the total supply for NYC V1[^3] and NYC V2[^4]
- add the total supplies accounting for decimal differences
  - NYC V1 has 0 decimals
  - NYC V2 has 6 decimals
- get the balance of the redemption contract itself
- set variables representing all data gathered above
- create a redemption ratio from `balance / total supply`
- set a variable that redemption is enabled, allowing access

The redemption extension then exposes a public function that:

- calculates amount to transfer based on `balance * redemption ratio`
- verify that redemption is enabled
- verify there is something to burn (NYC V1 or V2)
- verify there is something to redeem
- burn the NYC balance for the user
- transfer the redemption STX to the user from the contract
- store redemption info for the user and the contract
- print the redemption info from the read only functions

## Backwards Compatibility

This CCIP is supplemental to CCIP-020[^1] and not backwards compatible.

## Activation

This CCIP will be voted on using a vote contract that adheres to CCIP-015[^5] using the last two active stacking cycles for the protocol.

- MIA cycles 82 and 83
- NYC cycles 82 and 83

No scale factor is required.

Voting will begin when the contract is deployed and continue until the proposal is executed.

This CCIP follows the same voting format as CCIP-020[^1] except only NYC votes are counted.

- the votes are tallied and available in read-only functions
- the proposal will not pass unless 25% of NYC have participated

## Reference Implementations

- CCIP-022 Proposal: ccip022-citycoins-treasury-redemption-nyc.clar[^6]
- CCD-012 Redemption Extension: ccd012-redemption-nyc.clar[^7]

## Footnotes

[^1]: https://github.com/citycoins/governance/blob/main/ccips/ccip-020/ccip-020-graceful-protocol-shutdown.md
[^2]: https://explorer.hiro.so/txid/SP8A9HZ3PKST0S42VM9523Z9NV42SZ026V4K39WH.ccd002-treasury-nyc-mining-v2?chain=mainnet
[^3]: https://explorer.hiro.so/txid/SP2H8PY27SEZ03MWRKS5XABZYQN17ETGQS3527SA5.newyorkcitycoin-token?chain=mainnet
[^4]: https://explorer.stacks.co/txid/SPSCWDV3RKV5ZRN1FQD84YE1NQFEDJ9R1F4DYQ11.newyorkcitycoin-token-v2?chain=mainnet
[^5]: https://github.com/citycoins/governance/blob/main/ccips/ccip-015/ccip-015-community-proposal-voting-process.md
[^6]: https://github.com/citycoins/protocol/blob/fix/implement-ccip-022/contracts/proposals/ccip022-citycoins-treasury-redemption-nyc.clar
[^7]: https://github.com/citycoins/protocol/blob/fix/implement-ccip-022/contracts/extensions/ccd012-redemption-nyc.clar
