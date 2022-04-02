# CCIP-009

## Preamble

| CCIP Number   | 009                                   |
| ------------- | ------------------------------------- |
| Title         | CityCoins VRF v2                      |
| Author(s)     | Jason Schrader jason@joinfreehold.com |
| Consideration | Technical                             |
| Type          | Standard                              |
| Status        | Draft                                 |
| Created       | 2022-03-25                            |
| License       | BSD-2-Clause                          |
| Replaces      | CCIP-006                              |

## Introduction

The Stacks blockchain is a Layer 1 blockchain connected to Bitcoin, in which miners spend Bitcoin to bid for and win a fixed amount of Stacks tokens. Stackers have the option to lock up Stacks tokens for a specified amount of time, and in turn, receive a portion of the Bitcoin spent by miners proportionate to the amount Stacked[^1].

Taking this concept a level further, a new token (a "CityCoin") is created on the Stacks blockchain, following the SIP-010 fungible token standard[^2], such that CityCoins can be mined and Stacked per the methods stated above except a portion of the miner's bid would be redirected to the city's wallet overseen by a trusted third party custodian.

CityCoins leverage similar properties from the Proof of Transfer (PoX) consensus mechanism[^3] of the Stacks blockchain, programmed through a smart contract in Clarity.

This proposal describes the verifiable random function (VRF) used to calculate the block winner for a CityCoin.

## Specification

The VRF contract is used to determine the winner of a mined block.

This version introduces two functions to replace `get-random-uint-at-block` in CCIP-006:

- `get-rnd`: a read-only function that returns the random integer
- `get-save-rnd`: a public function that saves the random integer to a map, then returns it

Through the `get-rnd` function, the contract:

- checks the `BlockRnd` map to see if the random integer has been saved
  - if yes, returns the saved random integer
  - if no, it then:
    - fetches the on-chain VRF proof for the given block height
    - converts the lower 16 bytes into a uint
    - returns the random integer

Through the `get-save-rnd` function, the contract:

- checks the `BlockRnd` map to see if the random integer has been saved
  - if yes, returns the saved random integer
  - if no, it then:
    - fetches the on-chain VRF proof for the given block height
    - converts the lower 16 bytes into a uint
    - saves the random integer to the `BlockRnd` map
    - returns the random integer

The public function requires a private key and a Stacks transaction fee in order to send the transaction, and would ideally be used by any smart contract using this implementation to obtain a random value at a given block height.

More information on Stacks block assembly and the VRF proof can be found in [SIP-005](https://github.com/stacksgov/sips/blob/main/sips/sip-005/sip-005-blocks-and-transactions.md#blocks).

### Cost Savings

Using the Clarinet Console, we can evaluate the cost savings of the CCIP-009 VRF implementation.

#### Get Random Value

v1 VRF `get-random-uint-at-block`

```
+----------------------+----------+------------+------------+
|                      | Consumed | Limit      | Percent    |
+----------------------+----------+------------+------------+
| Runtime              | 390223   | 5000000000 | 0.008%     |
+----------------------+----------+------------+------------+
| Read count           | 4        | 7750       | 0.052%     |
+----------------------+----------+------------+------------+
| Read length (bytes)  | 3372     | 100000000  | 0.003%     |
+----------------------+----------+------------+------------+
| Write count          | 0        | 7750       | 0.000%     |
+----------------------+----------+------------+------------+
| Write length (bytes) | 0        | 15000000   | 0.000%     |
+----------------------+----------+------------+------------+
```

v2 VRF `get-save-rnd`, on first call before value is saved

```
+----------------------+----------+------------+------------+-----------+
|                      | Consumed | Limit      | Percent    | Diff (v1) |
+----------------------+----------+------------+------------+-----------+
| Runtime              | 59665    | 5000000000 | 0.001%     | -0.007%   |
+----------------------+----------+------------+------------+-----------+
| Read count           | 6        | 7750       | 0.077%     | +0.025%   |
+----------------------+----------+------------+------------+-----------+
| Read length (bytes)  | 2221     | 100000000  | 0.002%     | -0.001%   |
+----------------------+----------+------------+------------+-----------+
| Write count          | 1        | 7750       | 0.013%     | +0.013%   |
+----------------------+----------+------------+------------+-----------+
| Write length (bytes) | 33       | 15000000   | 0.000%     | +0.000%   |
+----------------------+----------+------------+------------+-----------+
```

v2 VRF `get-save-rnd`, on subsequent calls after value is saved

```
+----------------------+----------+------------+------------+-----------+
|                      | Consumed | Limit      | Percent    | Diff (v1) |
+----------------------+----------+------------+------------+-----------+
| Runtime              | 4782     | 5000000000 | 0.000%     | -0.008%   |
+----------------------+----------+------------+------------+-----------+
| Read count           | 4        | 7750       | 0.052%     | same      |
+----------------------+----------+------------+------------+-----------+
| Read length (bytes)  | 2220     | 100000000  | 0.002%     | -0.001%   |
+----------------------+----------+------------+------------+-----------+
| Write count          | 0        | 7750       | 0.000%     | same      |
+----------------------+----------+------------+------------+-----------+
| Write length (bytes) | 0        | 15000000   | 0.000%     | same      |
+----------------------+----------+------------+------------+-----------+
```

#### Claim Mining Reward

v1 `claim-mining-reward`

```
+----------------------+----------+------------+------------+
|                      | Consumed | Limit      | Percent    |
+----------------------+----------+------------+------------+
| Runtime              | 527214   | 5000000000 | 0.011%     |
+----------------------+----------+------------+------------+
| Read count           | 32       | 7750       | 0.413%     |
+----------------------+----------+------------+------------+
| Read length (bytes)  | 58176    | 100000000  | 0.058%     |
+----------------------+----------+------------+------------+
| Write count          | 5        | 7750       | 0.065%     |
+----------------------+----------+------------+------------+
| Write length (bytes) | 459      | 15000000   | 0.003%     |
+----------------------+----------+------------+------------+
```

v2 `claim-mining-reward` on first call before value is saved

```
+----------------------+----------+------------+------------+-----------+
|                      | Consumed | Limit      | Percent    | Diff (v1) |
+----------------------+----------+------------+------------+-----------+
| Runtime              | 196647   | 5000000000 | 0.004%     | -0.007%   |
+----------------------+----------+------------+------------+-----------+
| Read count           | 34       | 7750       | 0.439%     | +0.026%   |
+----------------------+----------+------------+------------+-----------+
| Read length (bytes)  | 57016    | 100000000  | 0.057%     | -0.001%   |
+----------------------+----------+------------+------------+-----------+
| Write count          | 6        | 7750       | 0.077%     | +0.012    |
+----------------------+----------+------------+------------+-----------+
| Write length (bytes) | 492      | 15000000   | 0.003%     | same      |
+----------------------+----------+------------+------------+-----------+
```

v2 `claim-mining-reward` on subsequent calls after value is saved

```
+----------------------+----------+------------+------------+-----------+
|                      | Consumed | Limit      | Percent    | Diff (v1) |
+----------------------+----------+------------+------------+-----------+
| Runtime              | 143482   | 5000000000 | 0.003%     | -0.008%   |
+----------------------+----------+------------+------------+-----------+
| Read count           | 32       | 7750       | 0.413%     | same      |
+----------------------+----------+------------+------------+-----------+
| Read length (bytes)  | 57047    | 100000000  | 0.057%     | -0.001%   |
+----------------------+----------+------------+------------+-----------+
| Write count          | 5        | 7750       | 0.065%     | same      |
+----------------------+----------+------------+------------+-----------+
| Write length (bytes) | 459      | 15000000   | 0.003%     | same      |
+----------------------+----------+------------+------------+-----------+
```

### Verification

Using the Clarinet Console, we can evaluate the responses side-by-side from each contract to ensure accuracy.

```
>> (contract-call? .citycoin-vrf get-random-uint-at-block u150)
(some u145987011005865973404065296999542273879)
>> (contract-call? .citycoin-vrf-v2 get-save-rnd u150)
(ok u145987011005865973404065296999542273879)

>> (contract-call? .citycoin-vrf get-random-uint-at-block u200)
(some u208380493011466864727649209133199743254)
>> (contract-call? .citycoin-vrf-v2 get-save-rnd u200)
(ok u208380493011466864727649209133199743254)

>> (contract-call? .citycoin-vrf get-random-uint-at-block u250)
(some u94492931307886473587440802540444121043)
>> (contract-call? .citycoin-vrf-v2 get-save-rnd u250)
(ok u94492931307886473587440802540444121043)
```

Based on the data above, the original CCIP-006 implementation and the new CCIP-009 return the same result for a given block height.

## Backwards Compatibility

This CCIP replaces the VRF architecture in CCIP-006 and is not backwards compatible, however the original the original CCIP-006 implementation will remain available and returns the same result.

## Activation

This CCIP will be voted on using a vote contract that adheres to CCIP-011[^4].

## Reference Implementations

TODO: add reference implementations after activation and deployment

## Footnotes

[^1]: https://stacking.club
[^2]: https://github.com/stacksgov/sips/blob/main/sips/sip-010/sip-010-fungible-token-standard.md
[^3]: https://docs.stacks.co/understand-stacks/proof-of-transfer
[^4]: https://github.com/citycoins/governance/blob/feat/community-upgrade-1/ccips/ccip-011/ccip-011-citycoins-stacked-tokens-voting.md
