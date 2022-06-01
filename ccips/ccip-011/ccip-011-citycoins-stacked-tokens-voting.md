# CCIP-011

## Preamble

| CCIP Number   | 011                                   |
| ------------- | ------------------------------------- |
| Title         | CityCoins Stacked Tokens Voting       |
| Author(s)     | Jason Schrader jason@joinfreehold.com |
| Consideration | Technical, Governance                 |
| Type          | Standard                              |
| Status        | Ratified                              |
| Created       | 2022-03-25                            |
| License       | BSD-2-Clause                          |
| Replaces      | CCIP-006                              |

## Introduction

The Stacks blockchain is a Layer 1 blockchain connected to Bitcoin, in which miners spend Bitcoin to bid for and win a fixed amount of Stacks tokens. Stackers have the option to lock up Stacks tokens for a specified amount of time, and in turn, receive a portion of the Bitcoin spent by miners proportionate to the amount Stacked[^1].

Taking this concept a level further, a new token (a "CityCoin") is created on the Stacks blockchain, following the SIP-010 fungible token standard[^2], such that CityCoins can be mined and Stacked per the methods stated above except a portion of the miner's bid would be redirected to the city's wallet overseen by a trusted third party custodian.

CityCoins leverage similar properties from the Proof of Transfer (PoX) consensus mechanism[^3] of the Stacks blockchain, programmed through a smart contract in Clarity.

This proposal describes the on-chain vote method used to ratify CCIP-008, CCIP-009, and CCIP-010.

## Specification

The voting contract is designed similarly to the SIP-012 voting process[^4], in which the user's vote is equal to the amount of tokens stacked across a defined set of reward cycles.

### Voting Actions

Through the `vote-on-proposal` function, a principal can submit their vote for a defined proposal and the contract will query the core contract for their stacked tokens, then update and track the vote totals for the proposal and the individual principal.

Within the vote contract are variables to track:

- the vote proposal ID, if used for a single proposal
- the vote scale factor, used to perform fixed-point calculations
- the vote scale factor, used to equalize CityCoin votes
- the initialization state of the contract
- the start and end block heights for the vote
- the name, link, and GitHub commit hash of the related CCIPs
- the total vote count, totals per CityCoin, and total votes
- the vote, totals per CityCoin, and total vote for individual principals

When a principal submits a vote, the contract then:

- obtains the voter ID for the principal, or creates one if it does not already exist
- verifies the vote contract is initialized with start and end block heights
- verifies the vote occurs between the start and end block heights
- checks if the principal's voter record already exists
  - if yes:
    - verifies that the vote is different from the previous vote
    - updates the vote totals and voter record
  - if no:
    - gets the CityCoin vote total per CityCoin based on stacking data in the core contract
    - calculates the vote total
    - verifies the vote total is greater than zero
    - updates the vote totals and voter record
- prints the vote totals and voter record

### Voting Information

The vote contract contains several functions that return information about the voting process.

- `is-initialized`: returns if the start/end block heights are set
- `get-proposals`: returns the list of proposals being voted on
- `get-vote-blocks`: returns the start/end block heights, if set
- `get-vote-start-block`: returns the start block height, if set
- `get-vote-end-block`: returns the end block height, if set
- `get-vote-amount`: returns the total vote for a given principal
- `get-proposal-votes`: returns the vote totals for a given proposal
- `get-voter`: returns the voter principal for a given voter ID
- `get-voter-id`: returns the voter ID for a given principal
- `get-voter-info`: returns the vote totals for a given principal

### Voting Calculations

Included with this CCIP is a [supplemental spreadsheet](./ccip-011-0001-sample-vote-calculations.ods) that shows how the scale factor can be calculated across CityCoins and provides some sample voting data from the selected cycles.

## Backwards Compatibility

None, this is the initial implementation.

## Activation

This voting process will be used for ratifying CCIP-008, CCIP-009, and CCIP-010. Upon a successful vote this CCIP will be activated.

## Reference Implementations

- `citycoins-vote-v1` deployed on the Stacks mainnet, implementing the vote method described in this CCIP[^5]

The vote was enabled through a community-provided user interface.[^6]

## Footnotes

[^1]: https://stacking.club
[^2]: https://github.com/stacksgov/sips/blob/main/sips/sip-010/sip-010-fungible-token-standard.md
[^3]: https://docs.stacks.co/understand-stacks/proof-of-transfer
[^4]: https://sip012.xyz/
[^5]: https://explorer.stacks.co/txid/SP34FHX44NK9KZ8KJC08WR2NHP8NEGFTTT7MTH7XD.citycoins-vote-v1?chain=mainnet
[^6]: https://vote.minecitycoins.com
