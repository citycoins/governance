# CCIP-025

## Preamble

| CCIP Number   | 025                                   |
| ------------- | ------------------------------------- |
| Title         | Extend Direct Execute Sunset Period 3 |
| Author(s)     | Friedger Müffke mail@friedger.de      |
|               | Jason Schrader jason@joinfreehold.com |
| Consideration | Governance                            |
| Type          | Standard                              |
| Status        | Ratified                              |
| Created       | 2024-10-23                            |
| License       | BSD-2-Clause                          |
| Supplements   | CCIP-012, CCIP-017, CCIP-021          |

## Introduction

When the ccd001-direct-execute[^1] Clarity contract was deployed, a sunset height of approximately 6 months was set,
after which the extension will no longer be able to execute proposals. This functionality was extended as part of CCIP-017[^2] and CCIP-021[^3].

The current sunset height is 173,748. After that block height no proposal can be executed anymore. In preparation of CCIP-016[^4] and CCIP-023[^5], an extension is required now.

This CCIP intends to:

- implement the CCIP-015[^6] voting mechanism as part of a DAO proposal
- extend the sunset period by an additional 25,920 Stacks blocks, ending at Stacks block 199,668

## Specification

The proposal will call `set-sunset-block` in ccd001-direct-execute[^1] with the value of `199668`.

### Proposal Voting

Proposal voting will be performed using the method described in CCIP-015[^6].

Voting will begin when the contract is deployed and continue for a set number of blocks.

> Note: The end of the voting period needs to allow enough time to execute the proposal and extend the sunset period. This will need to occur before Stacks block 173,748.

Votes will be counted two ways in the contract: a general total of yes/no votes, and the total number of votes based on the amount of CityCoins stacked in cycles 82 and 83.

The voting code is part of the CCIP-025 smart contract proposal[^7], which tracks user votes between the voting period.

It also restricts the `execute` function in the proposal so that it cannot run unless:

- the total votes are > 0
- the yes votes > no votes

The criteria above are the same as CCIP-017[^2] and CCIP-021[^3].

## Backwards Compatibility

This CCIP is supplemental to CCIP-012[^8], CCIP-017[^2], and CCIP-021[^3]

## Activation

This CCIP will be voted on and activated using a vote contract that adheres to CCIP-015[^6] using the last two active cycles for the protocol.

- Only MIA votes will be counted.
- Stacking amounts from cycles 82 and 83 will be considered.
- The voting period will start at the block which the contract is deployed
- No scale factor is required.

The proposal will follow the same voting format as CCIP-022[^2] and CCIP-024[^9]:

- Votes will be tallied and available in read-only functions.
- The vote will be considered valid if at least 25% of the total MIA stacked in cycles 82 and 83 participates.

## Reference Implementations

- TBD

## Footnotes

[^1]: https://explorer.hiro.so/txid/SP8A9HZ3PKST0S42VM9523Z9NV42SZ026V4K39WH.ccd001-direct-execute?chain=mainnet
[^2]: https://github.com/citycoins/governance/blob/main/ccips/ccip-017/ccip-017-extend-direct-execute-sunset-period.md
[^3]: https://github.com/citycoins/governance/blob/main/ccips/ccip-021/ccip-021-extend-direct-execute-sunset-period-2.md
[^4]: https://github.com/citycoins/governance/pull/16
[^5]: https://github.com/citycoins/governance/pull/40
[^6]: https://github.com/citycoins/governance/blob/main/ccips/ccip-015/ccip-015-community-proposal-voting-process.md
[^7]: TBD: Link to PR implementation
[^8]: https://github.com/citycoins/governance/blob/main/ccips/ccip-012/ccip-012-stabilize-emissions-and-treasuries.md
[^9]: https://github.com/citycoins/governance/blob/main/ccips/ccip-024/ccip-024-miamicoin-community-signal-vote.md
