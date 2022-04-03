# CCIP-008

## Preamble

| CCIP Number   | 008                                     |
| ------------- | --------------------------------------- |
| Title         | CityCoins SIP-010 Token v2              |
| Author(s)     | Jason Schrader jason@joinfreehold.com   |
|               | Andre Serrano andre@novablock.io        |
|               | BowTiedMooneeb bowtiedmooneeb@gmail.com |
| Consideration | Technical, Economic                     |
| Type          | Standard                                |
| Status        | Activation-In-Progress                  |
| Created       | 2022-03-25                              |
| License       | BSD-2-Clause                            |
| Replaces      | CCIP-005                                |

## Introduction

The Stacks blockchain is a Layer 1 blockchain connected to Bitcoin, in which miners spend Bitcoin to bid for and win a fixed amount of Stacks tokens. Stackers have the option to lock up Stacks tokens for a specified amount of time, and in turn, receive a portion of the Bitcoin spent by miners proportionate to the amount Stacked[^1].

Taking this concept a level further, a new token (a "CityCoin") is created on the Stacks blockchain, following the SIP-010 fungible token standard[^2], such that CityCoins can be mined and Stacked per the methods stated above except a portion of the miner's bid would be redirected to the city's wallet overseen by a trusted third party custodian.

CityCoins leverage similar properties from the Proof of Transfer (PoX) consensus mechanism[^3] of the Stacks blockchain, programmed through a smart contract in Clarity.

This proposal describes the token functionality for a CityCoin.

## Specification

### Token Configuration

CityCoins are fungible tokens on the Stacks blockchain that adhere to the SIP-010 fungible token standard[^2]. This includes specific functions defined in the SIP for transfer, name, symbol, decimals, balances, total supply, and token URI.

CityCoins have six decimal places, where one million micro-CityCoins (e.g. uMIA) is equal to one CityCoin (e.g. MIA).

The token URI can only be updated from the auth contract.

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

Miners receive coinbase rewards for mining a CityCoin outlined in the table below. The "halvings" are based on a doubling epoch model with an initial bonus period, in which the length of each epoch is twice as long as the last following epoch 1.

Following the token activation, the emissions schedule continues as follows:

| Epoch | Epoch Length | Epoch End Block | Block Reward | Total Supply   |
| ----- | ------------ | --------------- | ------------ | -------------- |
| 0     | 10,000       | 10,000          | 250,000      | 2,500,000,000  |
| 1     | 25,000       | 35,000          | 100,000      | 5,000,000,000  |
| 2     | 50,000       | 85,000          | 50,000       | 7,500,000,000  |
| 3     | 100,000      | 185,000         | 25,000       | 10,000,000,000 |
| 4     | 200,000      | 385,000         | 12,500       | 12,500,000,000 |
| 5     | 400,000      | 785,000         | 6,250        | 15,000,000,000 |
| 6     | 800,000      | 1,585,000       | 3,125        | 17,500,000,000 |

Note: the values above are denoted in CityCoins, and the values in the contract and APIs the value will represent the reward/supply above multiplied by `1,000,000` to account for the 6 decimal places as micro-CityCoins.

Included with this CCIP is a [supplemental spreadsheet](./ccip-008-0001-doubling-epoch-emissions-schedule.ods) that shows the calculations for and graphs the differences between the old and new emissions schedule.

## Backwards Compatibility

This CCIP replaces the token standard in CCIP-005 and is not backwards compatible, however any existing CityCoin deployed based on CCIP-005 can be converted to the equivalent CityCoin deployed based on procedures set forth in this CCIP.

The implementation needs to occur before Stacks block 59,497 in order for the total supply to be in sync with future CityCoins.

### Token Conversion

The CCIP-005 token contracts for MIA[^4] and NYC[^5] are currently deployed to the Stacks mainnet.

The CCIP-008 token contracts will be deployed to the Stacks mainnet and replace the functionality of the CCIP-005 token contracts.

In order to make the upgrade as seamless as possible, contract functions will be provided that:

- allow direct conversion by burning the CCIP-005 asset and minting the CCIP-008 asset
- allow conversion through mining and stacking within the newly deployed core contract
  - this will be available for a set number of blocks due to increased costs
  - once the end block height passes, the direct conversion will remain available

In addition, the activation block height and coinbase thresholds will be preserved for the original CCIP-005 token contracts, so that all emissions schedule between current and future CityCoins will be synchronized.

## Activation

This CCIP will be voted on using a vote contract that adheres to CCIP-011[^6].

## Reference Implementations

TODO: add reference implementations after activation and deployment

## Footnotes

[^1]: https://stacking.club
[^2]: https://github.com/stacksgov/sips/blob/main/sips/sip-010/sip-010-fungible-token-standard.md
[^3]: https://docs.stacks.co/understand-stacks/proof-of-transfer
[^4]: https://explorer.stacks.co/txid/SP466FNC0P7JWTNM2R9T199QRZN1MYEDTAR0KP27.miamicoin-token?chain=mainnet
[^5]: https://explorer.stacks.co/txid/SP2H8PY27SEZ03MWRKS5XABZYQN17ETGQS3527SA5.newyorkcitycoin-token?chain=mainnet
[^6]: https://github.com/citycoins/governance/blob/feat/community-upgrade-1/ccips/ccip-011/ccip-011-citycoins-stacked-tokens-voting.md
