# CCIP-021

## Preamble

| CCIP Number   | 021                                   |
| ------------- | ------------------------------------- |
| Title         | Extend Direct Execute Sunset Period 2 |
| Author(s)     | Friedger Müffke mail@friedger.de      |
| Consideration | Governance                            |
| Type          | Standard                              |
| Status        | Created                               |
| Created       | 2024-04-04                            |
| License       | BSD-2-Clause                          |
| Supplements   | CCIP-012                              |

## Introduction

When the ccd001-direct-execute[^1] Clarity contract was deployed, a sunset height of approximately 6 months was set,
after which the extension will no longer be able to execute proposals. More information can be found in issue 26 of the governance repository.[^2].

The current sunset height is 147,828. After that block height no proposal can be executed anymore. In preparation of CCIP-020[^3], an extension is required now.

This CCIP intends to:

- implement the CCIP-015[^4] voting mechanism as part of a DAO proposal
- extend the sunset period by an additional 25,920 Stacks blocks, ending at Stacks block 173,748

## Specification

The proposal will call `set-sunset-block` in ccd001-direct-execute[^1] with the value of `173748`.

### Proposal Voting

Proposal voting will be performed using the method described in CCIP-015[^4].

Voting will begin when the contract is deployed and continue for a set number of blocks.

> Note: The end of the voting period needs to allow enough time to execute the proposal and extend the sunset period. This will need to occur before Stacks block 147,828.

Votes will be counted two ways in the contract: a general total of yes/no votes, and the total number of votes based on the amount of CityCoins stacked in cycles 80 and 81.

The voting code is part of the CCIP-021 smart contract proposal[^5], which tracks user votes between the voting period.

It also restricts the `execute` function in the proposal so that it cannot run unless:

- the vote is concluded
- the total votes are > 0
- the yes votes > no votes

The criteria above are the same as CCIP-017[^6].

## Backwards Compatibility

This CCIP is supplemental to CCIP-012[^7] and CCIP-017[^6].

## Activation

This CCIP will be voted on and activated using a vote contract that adheres to CCIP-015[^4] using the last two active cycles from when the contract is deployed.

Currently, this would be:

- MIA cycles 80 and 81
- NYC cycles 80 and 81

The scale factor for MIA was determined using the same formula used in CCIP-015[^4] and calculated based on the total supply at the start block of the first cycle and the end block of the last cycle.

- MIA scale factor: 0.8916 _(prev: 0.8823)_

The calculations used for the scale factor are available in the supplemental spreadsheet[^8].

## Reference Implementations

- ccip-021-extend-sunset-period-2[^5]

## Footnotes

[^1]: https://explorer.hiro.so/txid/SP8A9HZ3PKST0S42VM9523Z9NV42SZ026V4K39WH.ccd001-direct-execute?chain=mainnet
[^2]: https://github.com/citycoins/governance/issues/26
[^3]: https://github.com/citycoins/governance/blob/feat/add-ccip-020/ccips/ccip-020/ccip-020-graceful-protocol-shutdown.md
[^4]: https://github.com/citycoins/governance/blob/feat/add-ccip-015/ccips/ccip-015/ccip-015-community-proposal-voting-process.md
[^5]: https://github.com/citycoins/governance/blob/feat/ccip-21/ccips/ccip-021/ccip-021-extend-direct-execute-sunset-period-2.md
[^6]: https://github.com/citycoins/governance/blob/feat/add-ccip-017/ccips/ccip-017/ccip-017-extend-direct-execute-sunset-period
[^7]: https://github.com/citycoins/governance/blob/main/ccips/ccip-012/ccip-012-stabilize-emissions-and-treasuries.md
[^8]: See the [ccip-021-vote-calculations-per-ccip-015 spreadsheet](./ccip-021-vote-calculations-per-ccip-015.ods)
