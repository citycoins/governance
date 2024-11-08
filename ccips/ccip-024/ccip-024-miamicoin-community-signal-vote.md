# CCIP-024

## Preamble

| CCIP Number   | 024                                   |
| ------------- | ------------------------------------- |
| Title         | MiamiCoin Community Signal Vote       |
| Author(s)     | Jason Schrader jason@joinfreehold.com |
| Consideration | Governance                            |
| Type          | Standard                              |
| Status        | Ratified                              |
| Created       | 2024-08-01                            |
| License       | BSD-2-Clause                          |
| Supplements   | CCIP-020, CCIP-022, CCIP-019          |

## Introduction

Following the execution of CCIP-020 Graceful Protocol Shutdown[^1] and CCIP-022 CityCoins Treasury Redemption NYC[^2], the MiamiCoin (MIA) community seeks to send a verifiable poll to the City of Miami regarding the STX treasury held by the protocol. This CCIP proposes an on-chain vote for MIA holders to express their desire for the City of Miami to consider nullifying their gift agreement and allow token holders to liquidate the treasury in a manner similar to the NYC redemption process outlined in CCIP-022.

## Specification

### Voting Mechanism

This CCIP will implement an on-chain vote using the method described in CCIP-015[^3], with the following specifications:

1. Only MiamiCoin (MIA) holders will be eligible to participate in the vote.
2. The vote will consider the stacking amounts for users in cycles 82 and 83.
3. The voting period will run for two weeks (approximately 2016 Bitcoin blocks) starting from the contract deployment.
4. The vote will have a defined end block based on the deployment block height.
5. The voting process will follow the same structure as CCIP-022 for a single city vote.

### Voting Contract

The voting contract will:

- Record the start and end block heights for the voting period.
- Calculate and store the total MIA stacked by each user in cycles 82 and 83.
- Allow users to cast their vote (Yes/No) during the voting period.
- Tally and store the total votes and voting power for each option.
- Provide read-only functions to access voting results and individual voter information.

### Voting Outcome

The result of this vote will serve as an officially verifiable message to the City of Miami, for consideration, expressing the community's desire regarding the STX treasury. It's important to note that this vote does not have any direct executable actions on the protocol itself.

## Backwards Compatibility

This CCIP is supplemental to CCIP-020[^1] and CCIP-022[^2] and is not backwards compatible.

## Activation

This CCIP will be implemented as an on-chain vote using a contract that adheres to CCIP-015[^3] with the following parameters:

- Only MIA votes will be counted.
- Stacking amounts from cycles 82 and 83 will be considered.
- The voting period will start at the contract deployment block and run for 2016 Bitcoin blocks (approximately two weeks).
- The end block will be calculated as: `start_block + 2016`.
- No scale factor is required.

The proposal will follow the same voting format as CCIP-022[^2]:

- Votes will be tallied and available in read-only functions.
- The vote will be considered valid if at least 25% of the total MIA stacked in cycles 82 and 83 participates.

## Reference Implementations

- CCIP-024 Proposal: ccip024-miamicoin-community-signal-vote.clar[^4]

## Footnotes

[^1]: https://github.com/citycoins/governance/blob/main/ccips/ccip-020/ccip-020-graceful-protocol-shutdown.md
[^2]: https://github.com/citycoins/governance/blob/main/ccips/ccip-022/ccip-022-citycoins-treasury-redemption-nyc.md
[^3]: https://github.com/citycoins/governance/blob/main/ccips/ccip-015/ccip-015-community-proposal-voting-process.md
[^4]: https://github.com/citycoins/protocol/blob/fix/implement-ccip-024/contracts/proposals/ccip024-miamicoin-signal-vote.clar
