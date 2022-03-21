# CCIP-004

## Preamble

| CCIP Number   | 004                                   |
| ------------- | ------------------------------------- |
| Title         | CityCoins Stacking                    |
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

This proposal describes the stacking functionality for a CityCoin.

## Specification

### Stacking

Anyone can Stack CityCoins by temporarily locking them in a CityCoins smart contract for a selected number of reward cycles. By doing so, stackers receive a portion of the remaining 70% of the STX sent by miners.

Stacking occurs over cycles that are 2,100 Stacks blocks in length, starting from the activation block height of the CityCoin contract.

More information about Activation can be found in [CCIP-002](../ccip-002/ccip-002-citycoins-activation.md).

Through the `stack-tokens` function, a principal can submit:

- the amount of CityCoins to be stacked
- the number of reward cycles to participate in

A stacker can participate in up to a maximum of 32 reward cycles at a time. Using the average block time of ten minutes per block, a single reward cycle is about two weeks in length and the maximum lock-up time is about sixteen months.

Stacking always begins in the next reward cycle, and after the stacking cycles are complete the principal will be ineligible to stack for one cycle, or a "cooldown period".

Stacked CityCoins are transferred to the contract, and can be reclaimed when the selected number of reward cycles have passed.

Within the core contract are variables to track:

- the maximum amount of reward cycles
- a list used as the reward cycle index
- how long a reward cycle is in Stacks blocks
- the stacking statistics for a given cycle
- the stacker statistics for a given cycle and user ID

When a principal submits CityCoins for stacking, the contract then:

- obtains the user ID for the principal, or creates one if it does not already exist
- verifies the contract is activated
- verifies the amount of CityCoins is greater than 0
- transfers the CityCoins to the contract
- updates the stacking statistics and stacker statistics for the number of cycles selected

### Stacking Claims

After a stacking cycle has completed a principal can claim the stacking rewards from that cycle, and if the total cycles selected are complete, the principal can reclaim their CityCoins.

The amount of STX a principal can claim is determined during a given cycle based on the amount stacked by the principal `R`, the total STX reward `S`, and the total stacked by all principals `T` using the formula:

`STX Rewards = (R * S) / T`

Through the `get-entitled-stacking-reward` function, the value of uSTX the principal can claim is returned if:

- the current block height is in a subsequent reward cycle
- the stacker actually locked up tokens in the target reward cycle
- the stacker locked up _enough_ tokens to get at least one uSTX

It is possible to Stack CityCoins and not receive uSTX if:

- no miners commit during this reward cycle
- the amount stacked by user is too few that you'd be entitled to less than 1 uSTX

Through the `get-stacker-at-cycle` function, the amount of CityCoins the principal can claim is returned as the `toReturn` value.

Through the `claim-stacking-reward` function, a principal can submit the target cycle to claim, and the contract will:

- verify that the reward cycle is complete, or continue if contract is shut down
- verify there are either STX or CityCoins to claim
- updates the stacker statistics to clear the values
- transfers the eligible STX and CityCoins to the principal

## Backwards Compatibility

None, as this proposal is the initial implementation.

## Activation

None, as this proposal is the initial implementation.

## Reference Implementations

MiamiCoin Contracts:

- `miamicoin-core-v1` deployed on the Stacks mainnet[^4], implementing:
  - the `get-stacker-at-cycle` function on lines 574-576
  - the `get-entitled-stacking-reward` function on lines 613-642
  - the `stack-tokens` function on lines 646-654
  - the `claim-stacking-reward` function on lines 750-756

NewYorkCityCoin Contracts:

- `newyorkcitycoin-core-v1` deployed on the Stacks mainnet[^5], implementing:
  - the `get-stacker-at-cycle` function on lines 579-581
  - the `get-entitled-stacking-reward` function on lines 618-647
  - the `stack-tokens` function on lines 651-659
  - the `claim-stacking-reward` function on lines 759-765

## Footnotes

[^1]: https://stacking.club
[^2]: https://github.com/stacksgov/sips/blob/main/sips/sip-010/sip-010-fungible-token-standard.md
[^3]: https://docs.stacks.co/understand-stacks/proof-of-transfer
[^4]: https://explorer.stacks.co/txid/SP466FNC0P7JWTNM2R9T199QRZN1MYEDTAR0KP27.miamicoin-core-v1?chain=mainnet
[^5]: https://explorer.stacks.co/txid/SP2H8PY27SEZ03MWRKS5XABZYQN17ETGQS3527SA5.newyorkcitycoin-core-v1?chain=mainnet
