# CCIP-019

## Preamble

| CCIP Number   | 019                                    |
| ------------- | -------------------------------------- |
| Title         | Stacks Nakomoto Release Preparation    |
| Author(s)     | Jason Schrader jason@joinfreehold.com  |
|               | Tim Butterfield tim@timbutterfield.com |
| Consideration | Governance, Economic, Technical        |
| Type          | Standard                               |
| Status        | Draft                                  |
| Created       | 2024-01-04                             |
| License       | BSD-2-Clause                           |
| Supplements   | CCIP-006, CCIP-012, CCIP-013, CCIP-014 |

## Introduction

With the upcoming Stacks Nakomoto release[^1], a new version of the PoX contract `.pox-4` will be released, which is used by the treasury contracts to perform delegated stacking.

On behalf of the community, based on discussion details in this GitHub issue[^2] and in CityCoins Discord channels, the community has voiced their interest in upgrading CityCoins in preparation for the Stacks Nakomoto release.

Based on suggestions from various community members, this CCIP intends to:

- implement the CCIP-015[^3] voting mechanism as part of a DAO proposal
- update .pox-3 to .pox-4 for delegated stacking in city treasury contracts

## Specification

### Treasury Contracts

The current treasury contracts used with mining directly call the `.pox-3` contract, which will be replaced with `.pox-4` in the upcoming Stacks 3.0 release[^1].

CCIP-019 will implement a new version of the treasury contract `ccd002-treasury-v3` that replaces `.pox-3` with `.pox-4`, as well as move the balances and perform the delegated stacking.

> Note: this only affects the mining treasuries for each city, the addresses for the stacking treasuries will remain the same

| City | Current                   | New                       |
| ---- | ------------------------- | ------------------------- |
| MIA  | ccd002-treasury-mining-v2 | ccd002-treasury-mining-v3 |
| MIA  | ccd002-treasury-stacking  | _(no change)_             |
| NYC  | ccd002-treasury-mining-v2 | ccd002-treasury-mining-v3 |
| NYC  | ccd002-treasury-stacking  | _(no change)_             |

### Proposal Voting

Proposal voting will be performed using the method described in CCIP-015[^3].

Voting will begin when the contract is deployed and continue for a set number of blocks.

Votes will be counted two ways in the contract: a general total of yes/no votes, and the total number of votes based on the amount of CityCoins stacked in cycles 82 and 83.

The voting code is part of the CCIP-019 smart contract proposal[^4], which tracks user votes between the voting period.

It also restricts the `execute` function in the proposal so that it cannot run unless:

- the vote is concluded
- the total votes are > 0
- the yes votes > no votes

## Backwards Compatibility

This CCIP is supplemental to CCIP-012[^5] and CCIP-013[^6].

## Activation

This CCIP will be voted on using a vote contract that adheres to CCIP-015[^3] using the last two active cycles from when the contract is deployed.

Currently, this would be:

- MIA cycles TBD and TBD
- NYC cycles TBD and TBD

The scale factor for MIA was determined using the same formula used in CCIP-015[^3] and calculated based on the total supply at the start block of the first cycle and the end block of the last cycle.

- MIA scale factor: TBD

The calculations used for the scale factor are available in the supplemental spreadsheet[^TBD].

## Reference Implementations

The reference implementation is published in the protocol repo[^4].

## Footnotes

[^1]: https://stacks.org/halving-on-horizon-nakamoto
[^2]: https://github.com/citycoins/governance/issues/23
[^3]: https://github.com/citycoins/governance/blob/main/ccips/ccip-015/ccip-015-community-proposal-voting-process.md
[^4]: https://github.com/citycoins/protocol/pull/67
