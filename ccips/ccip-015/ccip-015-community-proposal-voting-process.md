# CCIP-015

## Preamble

| CCIP Number   | 015                                   |
| ------------- | ------------------------------------- |
| Title         | Community Proposal Voting Process     |
| Author(s)     | Jason Schrader jason@joinfreehold.com |
|               | Tim Butterfield (TODO: email)         |
| Consideration | Governance                            |
| Type          | Standard                              |
| Status        | Draft                                 |
| Created       | 2023-04-26                            |
| License       | BSD-2-Clause                          |

## Introduction

This CCIP will implement a voting process for community members to participate in passing proposals executed by the DAO as defined in CCIP-012[^1].

It will follow the same voting mechanism described in CCIP-011[^2] adapted for use within a proposal.

## Specification

The voting contract is designed similarly to the SIP-012 voting process[^3], in which the user's vote is equal to the amount of tokens stacked across a defined set of reward cycles.

### Voting Trait

Any proposal using this CCIP will adhere to the `ccip-015-trait` defined below:

```clarity
(define-trait ccip-015-trait
	(
    (vote-on-proposal (bool) (response bool uint))
		(is-executable () (response bool uint))
	)
)
```

### Voting Actions

Through the `vote-on-proposal` function, a principal can submit their vote for a defined proposal and the contract will query the core contract for their stacked tokens, then update and track the vote totals for the proposal and the individual principal.

Within the vote contract are variables to track:

- the name, link, and GitHub commit hash of the related CCIP(s)
- the vote scale factor used to perform fixed-point calculations
- the vote scale factor used to equalize CityCoin votes
- the start and end block heights for the vote
- the total vote count, including totals per CityCoin and total votes
- the vote for individual participants, including the vote, totals per CityCoin, and total overall

When a principal submits a vote, the contract then:

- obtains the user ID for the principal, or fails if it does not exist
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

The proposal contract contains several functions that return information about the voting process.

- `is-executable`: returns true if the contract can be executed per the vote parameters
- `get-proposal-info`: returns the proposal information
- `get-vote-period`: returns the start/end blocks for the vote
- `get-vote-totals`: returns the vote totals for the proposal
- `get-voter-info`: returns the vote info for a given principal
- `get-mia-vote`: returns the MIA vote for a given city ID and user ID, optionally scaled
- `get-nyc-vote`: returns the NYC vote for a given city ID and user ID, optionally scaled

### Proposal Execution

The function `is-executable` defined above is used to prevent execution of the `execute` function that performs the actions defined in the proposal.

## Backwards Compatibility

This CCIP replaces CCIP-011[^2] and is supplemental to CCIP-012[^1].

## Activation

This CCIP will activate if CCIP-014[^4] successfully executes.

## Reference Implementations

- CCIP-014[^4]

## Footnotes

[^1]: TODO
[^2]: TODO
[^3]: TODO
[^4]: TODO
