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

With the upcoming Stacks Nakomoto release[^1], Stacks block times will no longer process at the same rate as Bitcoin blocks, reduced from an average of 10 minutes to 5 seconds.

CityCoins mining is based on Stacks blocks, and the issuance schedule in CCIP-012[^2] expects blocks to be produced at the rate of Bitcoin blocks, or on average every 10 minutes.

In addition, the Stacks Nakomoto release will include a new version of the PoX contract `.pox-4`, which is used by the treasury contracts to perform delegated stacking.

On behalf of the community, based on discussion details in this GitHub issue[^3] and in CityCoins Discord channels, the community has voiced their interest in upgrading CityCoins in preparation for the Stacks Nakomoto release.

Based on suggestions from various community members, this CCIP intends to:

- implement the CCIP-015[^3] voting mechanism as part of a DAO proposal
- implement the `burn-block-height` instead of the `block-height` for mining
- update .pox-3 to .pox-4 for delegated stacking in city treasury contracts

## Specification

### Mining Operations

The current mining process happens per Stacks block. To preserve the issuance schedule, the mining process will be updated to use the `burn-block-height` instead of the `block-height` for mining.

TODO: add more details

### Mining Treasury

The current mining contract directly references the mining treasury and cannot be updated.

CCIP-014 will implement a new version of the mining contract `ccd006-citycoin-mining-v3` that replaces the name with the new treasury described above.

At the block height the proposal executes, miners will need to switch to mining in the `ccd006-citycoin-mining-v3` contract.

Claims to the original and updated mining contract will function the same before and after.

### Treasury Contracts

The current treasury contracts used with mining directly call the `.pox-3` contract, which will be replaced with `.pox-4` in the upcoming Stacks 2.4 release[^1].

CCIP-014 will implement a new version of the treasury contract `ccd002-treasury-v3` that replaces `.pox-3` with `.pox-4`, as well as move the balances and perform the delegated stacking.

> Note: this only affects the mining treasuries for each city, the addresses for the stacking treasuries will remain the same

| City | Current                   | New                       |
| ---- | ------------------------- | ------------------------- |
| MIA  | ccd002-treasury-mining-v2 | ccd002-treasury-mining-v3 |
| MIA  | ccd002-treasury-stacking  | _(no change)_             |
| NYC  | ccd002-treasury-mining-v2 | ccd002-treasury-mining-v3 |
| NYC  | ccd002-treasury-stacking  | _(no change)_             |

### Proposal Voting

Proposal voting will be performed using the method described in CCIP-015[^TBD].

Voting will begin when the contract is deployed and continue for a set number of blocks.

Votes will be counted two ways in the contract: a general total of yes/no votes, and the total number of votes based on the amount of CityCoins stacked in cycles 54 and 55.

The voting code is part of the CCIP-019 smart contract proposal[^TBD], which tracks user votes between the voting period.

It also restricts the `execute` function in the proposal so that it cannot run unless:

- the vote is concluded
- the total votes are > 0
- the yes votes > no votes

## Backwards Compatibility

This CCIP is supplemental to CCIP-012[^TBD] and CCIP-013[^TBD].

## Activation

This CCIP will be voted on using a vote contract that adheres to CCIP-015[^TBD] using the last two active cycles from when the contract is deployed.

Currently, this would be:

- MIA cycles TBD and TBD
- NYC cycles TBD and TBD

The scale factor for MIA was determined using the same formula used in CCIP-015[^TBD] and calculated based on the total supply at the start block of the first cycle and the end block of the last cycle.

- MIA scale factor: TBD

The calculations used for the scale factor are available in the supplemental spreadsheet[^TBD].

## Reference Implementations

TBD

## Footnotes

[^1]: https://stacks.org/halving-on-horizon-nakamoto
[^2]: https://github.com/citycoins/governance/blob/main/ccips/ccip-012/
[^3]: https://github.com/citycoins/governance/issues/23
