# CCIP-017

## Preamble

| CCIP Number   | 017                                   |
| ------------- | ------------------------------------- |
| Title         | Extend Direct Execute Sunset Period   |
| Author(s)     | Jason Schrader jason@joinfreehold.com |
| Consideration | Governance                            |
| Type          | Standard                              |
| Status        | Ratified                              |
| Created       | 2023-08-31                            |
| License       | BSD-2-Clause                          |
| Supplements   | CCIP-012                              |

## Introduction

When the ccd001-direct-execute[^1] Clarity contract was deployed, a sunset height of 25,920 Stacks blocks or approximately 6 months was set, after which the extension will no longer be able to execute proposals. More information can be found in issue 17 of the governance repository.[^2].

As seen on the explorer[^1], the contract was deployed at Stacks block height 95,988 and will reach the sunset height at Stacks block 121,908.

This CCIP intends to:

- implement the CCIP-015[^3] voting mechanism as part of a DAO proposal
- extend the sunset period by an additional 25,920 Stacks blocks, ending at Stacks block 147,828

## Specification

The proposal will call `set-sunset-block` in ccd001-direct-execute[^1] with the value of `147828`.

### Proposal Voting

Proposal voting will be performed using the method described in CCIP-015[^3].

Voting will begin when the contract is deployed and continue for a set number of blocks.

> Note: The end of the voting period needs to allow enough time to execute the proposal and extend the sunset period. This will need to occur before Stacks block 121,908.

Votes will be counted two ways in the contract: a general total of yes/no votes, and the total number of votes based on the amount of CityCoins stacked in cycles 64 and 65.

The voting code is part of the CCIP-017 smart contract proposal[^4], which tracks user votes between the voting period.

It also restricts the `execute` function in the proposal so that it cannot run unless:

- the vote is concluded
- the total votes are > 0
- the yes votes > no votes

The criteria above are the same as CCIP-014[^5].

## Backwards Compatibility

This CCIP is supplemental to CCIP-012[^6].

## Activation

This CCIP will be voted on and activated using a vote contract that adheres to CCIP-015[^3] using the last two active cycles from when the contract is deployed.

Currently, this would be:

- MIA cycles 64 and 65
- NYC cycles 64 and 65

The scale factor for MIA was determined using the same formula used in CCIP-015[^3] and calculated based on the total supply at the start block of the first cycle and the end block of the last cycle.

- MIA scale factor: 0.8823 _(prev: 0.8760)_

The calculations used for the scale factor are available in the supplemental spreadsheet[^7].

## Reference Implementations

- [ccip-017-extend-sunset-period](https://explorer.hiro.so/txid/SP8A9HZ3PKST0S42VM9523Z9NV42SZ026V4K39WH.ccip017-extend-sunset-period?chain=mainnet)

## Footnotes

[^1]: https://explorer.hiro.so/txid/SP8A9HZ3PKST0S42VM9523Z9NV42SZ026V4K39WH.ccd001-direct-execute?chain=mainnet
[^2]: https://github.com/citycoins/governance/issues/17
[^3]: https://github.com/citycoins/governance/blob/feat/add-ccip-015/ccips/ccip-015/ccip-015-community-proposal-voting-process.md
[^4]: https://github.com/citycoins/governance/blob/feat/add-ccip-017/ccips/ccip-017/ccip-017-extend-direct-execute-sunset-period
[^5]: https://github.com/citycoins/governance/blob/main/ccips/ccip-014/ccip-014-upgrade-to-pox3.md
[^6]: https://github.com/citycoins/governance/blob/main/ccips/ccip-012/ccip-012-stabilize-emissions-and-treasuries.md
[^7]: See the [ccip-017-vote-calculations-per-ccip-015 spreadsheet](./ccip-017-vote-calculations-per-ccip-015.ods)
