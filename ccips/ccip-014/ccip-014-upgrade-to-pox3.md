# CCIP-014

## Preamble

| CCIP Number   | 014                                                  |
| ------------- | ---------------------------------------------------- |
| Title         | Upgrade to pox3                                      |
| Author(s)     | Raphael R. Sierra rapha@fontainebleau-management.com |
|               | Jason Schrader jason@joinfreehold.com                |
|               | Tim Butterfield (TODO: email)                        |
| Consideration | Governance, Economic                                 |
| Type          | Standard                                             |
| Status        | Draft                                                |
| Created       | 2023-04-19                                           |
| License       | BSD-2-Clause                                         |

On behalf of the community, see discussion details in issue https://github.com/citycoins/governance/issues/13 and in Discord channels.

I am passing on an issue that has been raised in the community discussion. The community has voiced their interest in upgrading Citycoins to pox-3. Based on suggestions from various community members, a CCIP could be initiated to cover the following points:

- Migrate to .pox-3 for delegated stacking
- Implement the CCIP-011 style voting mechanism through a DAO proposal
- Provide retroactive payment of STX rewards for the stacking contract bug.

## Introduction

On behalf of the community, based on discussion details in this GitHub issue[^1] and in CityCoins Discord channels, the community has voiced their interest in upgrading CityCoins to use pox-3 with the Stacks 2.4 release[^2].

Based on suggestions from various community members, this CCIP intends to:

- implement the CCIP-015[^3] voting mechanism as part of a DAO proposal
- update .pox to .pox-3 for delegated stacking in city treasury contracts

## Specification

### Treasury

The current treasury contracts used with mining directly call the `.pox` contract, which was disabled in Stacks 2.1 and will be replaced with `.pox-3` in the upcoming Stacks 2.4 release[^2].

CCIP-014 will implement a new version of the treasury contract `ccd002-treasury-v2` that replaces `.pox` with `.pox-3`, as well as move the balances and perform the delegated stacking.

> Note: this only affects the mining treasuries for each city, the addresses for the stacking treasuries will remain the same

| City | Current                  | New                       |
| ---- | ------------------------ | ------------------------- |
| MIA  | ccd002-treasury-mining   | ccd002-treasury-mining-v2 |
| MIA  | ccd002-treasury-stacking | _(no change)_             |
| NYC  | ccd002-treasury-mining   | ccd002-treasury-mining-v2 |
| NYC  | ccd002-treasury-stacking | _(no change)_             |

### Mining

The current mining contract directly references the mining treasury and cannot be updated.

CCIP-014 will implement a new version of the mining contract `ccd006-citycoin-mining-v2` that replaces the name with the new treasury described above.

At the block height the proposal executes, miners will need to switch to mining in the `ccd006-citycoin-mining-v2` contract.

Claims to the original and updated mining contract will function the same before and after.

### Proposal Voting

Proposal voting will be performed using the method described in CCIP-015[^3].

Voting will begin when the contract is deployed and continue for a set number of blocks.

> Note: The end of the voting period needs to allow enough time to execute the proposal and stack the city treasuries. This will require close monitoring of the Stacks 2.4 launch plan.

Votes will be counted two ways in the contract: a general total of yes/no votes, and the total number of votes based on the amount of CityCoins stacked in cycles 54 and 55.

The voting code is part of the CCIP-014 smart contract proposal[^4], which tracks user votes between the voting period.

It also restricts the `execute` function in the proposal so that it cannot run unless:

- the vote is concluded
- the total votes are > 0
- the yes votes > no votes

The criteria above are in consideration of the short window to stack the treasuries for Cycle 60, and can be adjusted in future proposals.

## Backwards Compatibility

This CCIP is supplemental to CCIP-012[^5] and CCIP-013[^6].

## Activation

This CCIP will be voted on using a vote contract that adheres to CCIP-015[^3] using the last two active cycles from when the contract is deployed.

Currently, this would be:

- MIA cycles 54 and 55
- NYC cycles 54 and 55

The scale factor for MIA was determined using the same formula used in CCIP-015[^3] and calculated based on the total supply at the start block of the first cycle and the end block of the last cycle.

- MIA scale factor: 0.8760 _(prev: 0.8715)_

The calculations used for the scale factor are available in the supplemental spreadsheet[^7].

## Reference Implementations

- TODO

## Footnotes

[^1]: https://github.com/citycoins/governance/issues/13
[^2]: https://forum.stacks.org/t/stacks-2-4-is-here-stacking-to-be-re-enabled/15024
[^3]: https://github.com/citycoins/governance/blob/feat/add-ccip-015/ccips/ccip-015/ccip-015-community-proposal-voting-process.md
[^4]: https://github.com/citycoins/protocol/blob/fix/support-pox-2/contracts/proposals/ccip014-pox-3.clar
[^5]: https://github.com/citycoins/governance/blob/main/ccips/ccip-012/ccip-012-stabilize-emissions-and-treasuries.md
[^6]: https://github.com/citycoins/governance/blob/main/ccips/ccip-013/ccip-013-stabilize-protocol-and-simplify-contracts.md
[^7]: TODO: update after upload
