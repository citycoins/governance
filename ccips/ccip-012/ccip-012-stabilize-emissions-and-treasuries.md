# CCIP-012

## Preamble

| CCIP Number   | 012                                   |
| ------------- | ------------------------------------- |
| Title         | Stabilize Emissions and Treasuries    |
| Author(s)     | Jason Schrader jason@joinfreehold.com |
| Consideration | Economic, Governance, Technical       |
| Type          | Standard                              |
| Status        | Draft                                 |
| Created       | 2022-08-04                            |
| License       | BSD-2-Clause                          |
| Replaces      | TBD                                   |

## Introduction

This proposal implements the first of two changes to help stabilize the CityCoins protocol design, and allow for future development, experimentation, and growth.

Please see CCIP-013[^1] for the second part of the proposal.

This proposal is divided into two phases:

- Phase 1: reduce current CityCoin emissions to 2%
- Phase 2: move treasuries to smart contract vaults

## Specification

### Phase 1: Reduce CityCoin Emissions

Previously, the community successfully voted for and implemented CCIP-008 in April 2022[^2]. This change included a single iteration to test stemming the miner arbitrage problem, the results were mixed as the arbitrage volume was reduced, but is still persistent[^3].

Phase 1 change reduces inflation to 2% annually for existing CityCoins starting at Stacks block height ~~73,000~~, and works in concert with proposed changes described in Phase 2 and Phase 3 included in CCIP-013[^1].

TODO: add new block height above

![Comparison of CityCoins Inflation Rates](citycoins-annualized-inflation-rate-comparison.png)

To implement this change a proposal will be submitted to the auth contract for MIA/NYC, using the `update-coinbase-amounts` function defined in CCIP-010[^4].

The coinbase amounts will be updated based on the schedule described in the linked spreadsheet[^5], with a visual example of the reduction in total supply below:

TODO: put finalized spreadsheet in ODS format

TODO: add table with block reward comparisons

| Blocks Passed | V1 Total Supply | V2 Total Supply | V3 Total Supply |
| ------------- | --------------- | --------------- | --------------- |
| 10,000        | 2,500,000,000   | 2,500,000,000   | 2,500,000,000   |
| 30,000        | 4,500,000,000   | 4,500,000,000   | 4,500,000,000   |
| 50,000        | 6,500,000,000   | 5,750,000,000   | 5,750,000,000   |
| 100,000       | 11,500,000,000  | 7,875,000,000   | 5,859,398,782   |
| 200,000       | 21,500,000,000  | 10,187,500,000  | 6,082,359,162   |
| 400,000       | 32,000,000,000  | 12,593,750,000  | 6,545,247,987   |
| 800,000       | 40,375,000,000  | 15,046,875,000  | 7,541,480,558   |

### Phase 2: Move CityCoin Treasuries to Smart Contract Vaults

Under the current protocol the treasuries for a CityCoin are stored in a 2-of-3 multi-signature Bitcoin wallet.

Bitcoin wallets possess the ability to make STX transfers and stack STX via Bitcoin transactions, but cannot interact with smart contracts.

This phase would replace the 2-of-3 multi-signature wallet with a smart contract vault secured by a DAO implementation that can grow with the protocol.

The basic structure of the DAO would start with:

- deploying the initial contract (similar to ExecutorDAO[^6]/StackerDAOs[^7]/EcosystemDAO[^8])
- setup with 3-of-5 signers from the auth contract
- enable proposals and temporary veto/execution

Using this DAO structure, two proposals would be created and executed:

- create treasuries for the existing cities
- stack treasuries for the existing cities

The structure of the DAO is very flexible and would provide an easy path to implementing features such as community proposals, community voting (via CCIP-011[^9] or a new method), as well as directly manage the protocol contracts through the DAO.

## Backwards Compatibility

TODO: expand on compatibility

- CCIP-008 Token v2
- CCIP-010 Auth v2

## Activation

This CCIP will be voted on using a vote contract that adheres to CCIP-011[^9] using the last two active cycles from when the contract is deployed.

## Reference Implementations

TODO: add vote contract
TODO: add before/after snapshot of data

## Footnotes

TODO: update footnote links before merge

[^1]: https://github.com/citycoins/governance/blob/feat/stabilize-protocol/ccips/ccip-013/ccip-013-stabilize-protocol-and-simplify-contracts.md
[^3]: https://vote.minecitycoins.com/
[^4]: https://docs.google.com/spreadsheets/d/1d3IPNPENA1GOVFCB25tehUw14t_y4yzpU0X3euoCmUc/edit#gid=148425312
[^5]: https://github.com/citycoins/governance/blob/main/ccips/ccip-010/ccip-010-citycoins-auth-v2.md
[^6]: https://docs.google.com/spreadsheets/u/1/d/17aF6LlniJ3Bk3YomDynOHRrx4vxiyotplD42hUbiSRA/edit#gid=1451657949
[^7]: https://github.com/MarvinJanssen/executor-dao
[^8]: https://github.com/StackerDAOs/backend
[^9]: https://github.com/Clarity-Innovation-Lab/ecosystem-dao
[^9]: https://github.com/citycoins/governance/blob/main/ccips/ccip-011/ccip-011-citycoins-stacked-tokens-voting.md
