# CityCoins Improvement Proposals

The CCIPs describe protocol-level changes for CityCoins.

Note: This is a WIP and more content will be added over the next week!

## Basic Structure

- [Issues](https://github.com/citycoins/governance/issues) - create and discuss a formal proposal to be voted on
- [Pull Requests](https://github.com/citycoins/governance/pulls) - officially record and vote on the proposal

Issues relate to discussions about a particular topic, likely coming from Discord.

Proposals, stored in a directory and numbered, represent well-formed proposed changes.

Proposals can then be voted for on-chain using a smart contract.

## Contract Voting

The contract would store proposal information, including a GitHub link and commit hash.

Tags could be created for each new proposal as well.

The contract would store total vote information as well as individual voter records.

Voting would run between designated block heights.

Addresses can only vote once, and their stacked amounts of MIA and NYC both count toward the total.

Addresses can reverse their vote, if desired, but cannot withdraw their vote.
