# CCIP-008

## Preamble

| CCIP Number   | 008                                   |
| ------------- | ------------------------------------- |
| Title         | CityCoins SIP-010 Token v2            |
| Author(s)     | Jason Schrader jason@joinfreehold.com |
| Consideration | Technical, Economic                   |
| Type          | Standard                              |
| Status        | Draft                                 |
| Created       | 2022-03-25                            |
| License       | BSD-2-Clause                          |
| Replaces      | CCIP-005                              |

## Introduction

The Stacks blockchain is a Layer 1 blockchain connected to Bitcoin, in which miners spend Bitcoin to bid for and win a fixed amount of Stacks tokens. Stackers have the option to lock up Stacks tokens for a specified amount of time, and in turn, receive a portion of the Bitcoin spent by miners proportionate to the amount Stacked[^1].

Taking this concept a level further, a new token (a "CityCoin") is created on the Stacks blockchain, following the SIP-010 fungible token standard[^2], such that CityCoins can be mined and Stacked per the methods stated above except a portion of the miner's bid would be redirected to the city's wallet overseen by a trusted third party custodian.

CityCoins leverage similar properties from the Proof of Transfer (PoX) consensus mechanism[^3] of the Stacks blockchain, programmed through a smart contract in Clarity.

This proposal describes the token functionality for a CityCoin.

## Specification

### Token Configuration

CityCoins are fungible tokens on the Stacks blockchain that adhere to the SIP-010 fungible token standard[^2]. This includes specific functions defined in the SIP for transfer, name, symbol, decimals, balances, total supply, and token URI.

CityCoins have 6 decimal places, and the token URI can only be updated from the auth contract.

CityCoins can be sent and received on the Stacks blockchain, interact with smart contracts, and be used by any interface that accepts a SIP-010 fungible token[^2].

CityCoins also follow the CityCoin Token Trait, defined in [CCIP-001](../ccip-001/ccip-001-citycoins-traits.md).

Within the token contract are variables to track:

- the number of Stacks blocks before a halving occurs
- the Stacks block thresholds set for each halving
- the activation status of the token contract
- the core contract states

### Token Activation

Once a CityCoin is deployed and activated, the emission schedule begins starting from the activation block height of the CityCoin core contract.

More information about Activation can be found in [CCIP-002](../ccip-002/ccip-002-citycoins-activation.md).

CityCoins have no ICO, no pre-sale, and no pre-mine. CityCoins are only mined into existence.

### Emissions Schedule

TODO: New Emissions Schedule
TODO: add spreadsheet references (ODS format)

## Backwards Compatibility

This CCIP replaces the token standard in CCIP-005 and is not backwards compatible.

TODO: Migration/Conversion

## Activation

TODO: On-chain vote description

## Reference Implementations

TODO: add references

## Footnotes

[^1]: https://stacking.club
[^2]: https://github.com/stacksgov/sips/blob/main/sips/sip-010/sip-010-fungible-token-standard.md
[^3]: https://docs.stacks.co/understand-stacks/proof-of-transfer
