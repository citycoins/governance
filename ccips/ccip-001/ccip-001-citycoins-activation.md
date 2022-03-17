# CCIP-001

## Preamble

| CCIP Number   | 001                                   |
| ------------- | ------------------------------------- |
| Title         | CityCoins Activation                  |
| Author(s)     | Jason Schrader jason@joinfreehold.com |
| Consideration | Technical                             |
| Type          | Standard                              |
| Status        | Draft                                 |
| Created       | 2022-03-16                            |
| License       | BSD-2-Clause                          |

## Introduction

This proposal describes the initial version of the CityCoins Protocol.

Bitcoin is finding utility on the balance sheet of several corporate treasuries as protection against monetary debasement and inflation[^1], and local officials are looking into use cases for cryptocurrency, generally starting with Bitcoin.

One challenge for a city mining Bitcoin to put it on their balance sheet is that it expensive to do and it does not produce additional earnings once the asset is mined. Mining Bitcoin is a competitive market dominated by large players, such that the capital requirement to start mining, the break-even point, and the power consumption are all considerations a city would need to resolve.

The Stacks blockchain is a Layer 1 blockchain connected to Bitcoin, in which miners spend Bitcoin to bid for and win a fixed amount of Stacks tokens. Stackers have the option to lock up Stacks tokens for a specified amount of time, and in turn, receive a portion of the Bitcoin spent by miners proportionate to the amount Stacked[^2].

Taking this concept a level further, a new token (a "CityCoin") is created on the Stacks blockchain, following the SIP-010 fungible token standard[^3], such that CityCoins can be mined and Stacked per the methods stated above except a portion of the miner's bid would be redirected to the city's wallet overseen by a trusted third party custodian.

CityCoins leverage similar properties from the Proof of Transfer (PoX) consensus mechanism[^4] of the Stacks blockchain, programmed through a smart contract in Clarity. The protocol operations are described in four categories for activation, mining, stacking, and the token. Further detail is provided for each category in the specification section.

[^1]: https://bitcointreasuries.org/
[^2]: https://stacking.club
[^3]: https://github.com/stacksgov/sips/blob/main/sips/sip-010/sip-010-fungible-token-standard.md
[^4]: https://docs.stacks.co/understand-stacks/proof-of-transfer

## Specification

### Activation

After a CityCoin contract is deployed, an additional activation step must occur before any additional functions become available.

A threshold is set within the contract through which anyone can send a transaction as a signal for activating the CityCoin.

Once that threshold is met, the current block height of the final transaction is added to the activation delay set in the contract, calculating the block height at which mining and stacking become available.

There are no CityCoins minted or distributed prior to the start of mining.

### Mining

### Stacking

## Related Work

## Backwards Compatibility

None, intial implementation-style.

## Activation

## Reference Implementations

MiamiCoin (MIA)

NewYorkCityCoin (NYC)
