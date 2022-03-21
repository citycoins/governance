# CCIP-005

## Preamble

| CCIP Number   | 005                                   |
| ------------- | ------------------------------------- |
| Title         | CityCoins SIP-010 Token               |
| Author(s)     | Jason Schrader jason@joinfreehold.com |
| Consideration | Technical, Economic                   |
| Type          | Standard                              |
| Status        | Draft                                 |
| Created       | 2022-03-16                            |
| License       | BSD-2-Clause                          |

## Introduction

The Stacks blockchain is a Layer 1 blockchain connected to Bitcoin, in which miners spend Bitcoin to bid for and win a fixed amount of Stacks tokens. Stackers have the option to lock up Stacks tokens for a specified amount of time, and in turn, receive a portion of the Bitcoin spent by miners proportionate to the amount Stacked[^1].

Taking this concept a level further, a new token (a "CityCoin") is created on the Stacks blockchain, following the SIP-010 fungible token standard[^2], such that CityCoins can be mined and Stacked per the methods stated above except a portion of the miner's bid would be redirected to the city's wallet overseen by a trusted third party custodian.

CityCoins leverage similar properties from the Proof of Transfer (PoX) consensus mechanism[^3] of the Stacks blockchain, programmed through a smart contract in Clarity.

This proposal describes the token functionality for a CityCoin.

## Specification

### Token Configuration

CityCoins are fungible tokens on the Stacks blockchain that adhere to the SIP-010 fungible token standard[^2]. This includes specific functions defined in the SIP for transfer, name, symbol, decimals, balances, total supply, and token URI.

CityCoins can be sent and received on the Stacks blockchain, interact with smart contracts, and be used by any interface that accepts a SIP-010 fungible token.

CityCoins also follow the CityCoin Token Trait, defined in [CCIP-001](../ccip-001/ccip-001-citycoins-traits.md).

### Token Activation

Once a CityCoin is deployed and activated, the emission schedule begins starting from the activation block height of the CityCoin core contract.

More information about Activation can be found in [CCIP-002](../ccip-002/ccip-002-citycoins-activation.md).

CityCoins have no ICO, no pre-sale, and no pre-mine. CityCoins are only mined into existence.

### Emissions Schedule

Miners receive coinbase rewards for mining the CityCoin outlined in the table below. The "halvings" occur at intervals similar to Bitcoin and Stacks, every 210,000 blocks, and there is a 10,000 block bonus reward for early miners.

The issuance schedule does not begin until mining is activated, and once it begins, the current block height of the Stacks blockchain is recorded in the contract. From there, the issuance continues as follows:

| Time Period                   | Reward            | Notes                               |
| ----------------------------- | ----------------- | ----------------------------------- |
| First 10,000 Stacks Blocks    | 250,000 CityCoins | approx. 3 months                    |
| Next 200,000 Stacks Blocks    | 100,000 CityCoins | approx. 4 years, minus bonus period |
| Next 210,000 Stacks Blocks    | 50,000 CityCoins  | approx. 4 years                     |
| Next 210,000 Stacks Blocks    | 25,000 CityCoins  | approx. 4 years                     |
| Next 210,000 Stacks Blocks    | 12,500 CityCoins  | approx. 4 years                     |
| Next 210,000 Stacks Blocks    | 6,250 CityCoins   | approx. 4 years                     |
| After 1,050,000 Stacks Blocks | 3,125 CityCoins   | continues indefinitely              |

After the final halving at 1,050,000 Stacks blocks past the Stacks block height recorded at activation, the total supply is estimated to be `42,187,500,000` and will increase indefinitely by `164,062,500` per year.

## Backwards Compatibility

None, as this proposal is the initial implementation.

## Activation

None, as this proposal is the initial implementation.

## Reference Implementations

- `miamicoin-token` deployed on the Stacks mainnet, implementing

## Footnotes

[^1]: https://stacking.club
[^2]: https://github.com/stacksgov/sips/blob/main/sips/sip-010/sip-010-fungible-token-standard.md
[^3]: https://docs.stacks.co/understand-stacks/proof-of-transfer
