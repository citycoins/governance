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

CityCoins mining occurs 1:1 with the production of Stacks blocks starting from the activation block height of the CityCoin core contract.

More information about Activation can be found in [CCIP-002](../ccip-002/ccip-002-citycoins-activation.md).

Through the `mine-tokens` function, a principal can submit:

- the amount of STX to commit
- an optional memo, printed on-chain

Through the `mine-many` function, a principal can submit a list of commits up to 200 in length, which are then applied to the selected number of blocks starting with the first block height of the successful transaction.

Miners compete against each other and the winner is selected by a Verifiable Random Function (VRF) weighted by the miner's individual contribution against the total contribution in that block.

Miners can only participate once per block, and STX sent to the contract for mining are not returned.

Within the core contract are variables to track:

- the percentage of the total STX that is sent to the city's reserved wallet
- the number of blocks a winning miner must wait to claim their reward
- the mining statistics for a given block
- the miner statistics for a given block and user ID
- the high value of the current block, that increments with each miner commitment
- the user ID of the miner that won the block

When a principal submits a commit to mine in a block, the contract then:

- obtains the user ID for the principal, or creates one if it does not already exist
- verifies the contract is activated and the principal has not mined in this block already
- verifies the amount of STX submitted is greater than zero and less than the principal's balance
- updates the mining statistics, miner statistics, and high value for the block
- If stacking is active, updates the stacking stats for the cycle
- distributes a percentage of the STX that miners forward directly to a reserved wallet for the city.
- distributes the remaining percentage in one of two ways:
  - If stacking is not active in the current reward cycle, it is sent to the city's reserved wallet.
  - If stacking is active in the current reward cycle, it is distributed to the addresses that temporarily lock ("stack") their CityCoins.

More information about Stacking can be found in [CCIP-004](../ccip-004/ccip-004-citycoins-stacking.md).

### Mining Claims

The winning miner can claim newly minted CityCoins at any time after the maturity window passes from the block height they won. The maturity window protects the VRF seed from being known in advance.

The amount of CityCoins the miner can claim is based on the emissions schedule and the block height they won.

Through the `claim-mining-reward` function, a principal can submit the block height they won, and if verified as the winner of the block the new CityCoins are minted to the caller.

When a principal submits a block to claim, the contract then:

- verifies the reward was not already claimed
- verifies the principal did win the block
- updates the mining statistics, miner statistics, and block winner
- submits the mint transaction to the token contract with the block height

New CityCoins are not minted until they are claimed.

## Backwards Compatibility

None, as this proposal is the initial implementation.

## Activation

None, as this proposal is the initial implementation.

## Reference Implementations

MiamiCoin Contracts:

- `miamicoin-core-v1` deployed on the Stacks mainnet[^4], implementing:
  - the `mine-tokens` function on lines 279-287
  - the `mine-many` function on lines 289-307
  - the `claim-mining-reward` function on lines 434-440

NewYorkCityCoin Contracts:

- `newyorkcitycoin-core-v1` deployed on the Stacks mainnet[^5], implementing:
  - the `mine-tokens` function on lines 280-288
  - the `mine-many` function on lines 290-312
  - the `claim-mining-reward` function on lines 439-445

## Footnotes

[^1]: https://stacking.club
[^2]: https://github.com/stacksgov/sips/blob/main/sips/sip-010/sip-010-fungible-token-standard.md
[^3]: https://docs.stacks.co/understand-stacks/proof-of-transfer
[^4]: https://explorer.stacks.co/txid/SP466FNC0P7JWTNM2R9T199QRZN1MYEDTAR0KP27.miamicoin-core-v1?chain=mainnet
[^5]: https://explorer.stacks.co/txid/SP2H8PY27SEZ03MWRKS5XABZYQN17ETGQS3527SA5.newyorkcitycoin-core-v1?chain=mainnet
