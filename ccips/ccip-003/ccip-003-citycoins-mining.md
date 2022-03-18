# CCIP-003

## Preamble

| CCIP Number   | 003                                   |
| ------------- | ------------------------------------- |
| Title         | CityCoins Mining                      |
| Author(s)     | Jason Schrader jason@joinfreehold.com |
| Consideration | Technical                             |
| Type          | Standard                              |
| Status        | Draft                                 |
| Created       | 2022-03-16                            |
| License       | BSD-2-Clause                          |

## Introduction

The Stacks blockchain is a Layer 1 blockchain connected to Bitcoin, in which miners spend Bitcoin to bid for and win a fixed amount of Stacks tokens. Stackers have the option to lock up Stacks tokens for a specified amount of time, and in turn, receive a portion of the Bitcoin spent by miners proportionate to the amount Stacked[^1].

Taking this concept a level further, a new token (a "CityCoin") is created on the Stacks blockchain, following the SIP-010 fungible token standard[^2], such that CityCoins can be mined and Stacked per the methods stated above except a portion of the miner's bid would be redirected to the city's wallet overseen by a trusted third party custodian.

CityCoins leverage similar properties from the Proof of Transfer (PoX) consensus mechanism[^3] of the Stacks blockchain, programmed through a smart contract in Clarity.

This proposal describes the mining functionality for a CityCoin.

## Specification

### Mining

Anyone can mine CityCoins by submitting STX into a CityCoins smart contract on the Stacks blockchain.

CityCoins mining occurs 1:1 with the production of Stacks blocks.

Through the `mine-tokens` function, a principal can submit:

- the amount of STX to bid
- an optional memo, written on-chain

Through the `mine-many` function, a principal can submit a list of bids up to 200 in length.

Miners compete against each other and the winner is selected by a Verifiable Random Function (VRF) weighted by the miner's individual contribution against the total contribution in that block.

Miners can only participate once per block, and STX sent to the contract for mining are not returned.

Within the core contract are variables to track:

- the percentage of the total STX that is sent to the city's reserved wallet
- the number of blocks a winning miner must wait to claim their reward
- the mining statistics for a given block
- the miner statistics for a given block and user ID
- the high value of the current block, that increments with each miner commitment
- the user ID of the miner that won the block

30% of the STX that miners forward is sent directly to a reserved wallet for the city.

The remaining 70% are distributed in one of two ways:

- If stacking is not active in the current reward cycle, it is sent to the city's reserved wallet.
- If stacking is active in the current reward cycle, it is distributed to the addresses that temporarily lock ("stack") their CityCoins.

More information about Stacking can be found in [CCIP-004](../ccip-004/ccip-004-citycoins-stacking.md).

### Mining Claims

The winning miner can claim newly minted CityCoins at any time after the maturity window passes from the block height they won.

The amount of CityCoins the miner can claim is based on the emissions schedule and the block height they won.

Through the `claim-mining-reward` function, a principal can submit the block height they won, and if verified as the winner of the block the new CityCoins are minted to the caller.

## Backwards Compatibility

None, as this proposal is the initial implementation.

## Activation

None, as this proposal is the initial implementation.

## Reference Implementations

MiamiCoin Contracts:

NewYorkCityCoin Contracts:

## Footnotes

[^1]: https://stacking.club
[^2]: https://github.com/stacksgov/sips/blob/main/sips/sip-010/sip-010-fungible-token-standard.md
[^3]: https://docs.stacks.co/understand-stacks/proof-of-transfer
