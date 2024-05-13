# CCIP-019

## Preamble

| CCIP Number   | 019                                              |
| ------------- | ------------------------------------------------ |
| Title         | PoX 4 Stacking                                   |
| Author(s)     | Jason Schrader jason@joinfreehold.com            |
|               | Tim Butterfield tim@timbutterfield.com           |
|               | Friedger Müffke mail@friedger.de                 |
| Consideration | Governance, Economic, Technical                  |
| Type          | Standard                                         |
| Status        | Draft                                            |
| Created       | 2024-01-04                                       |
| License       | BSD-2-Clause                                     |
| Supplements   | CCIP-006, CCIP-012, CCIP-013, CCIP-014, CCIP-020 |

## Introduction

With the Stacks Nakomoto release[^1], a new version of the PoX contract `.pox-4` was released. The previous versions of the PoX contracts were used by the treasury contracts to perform delegated stacking (STX). Currently, both CityCoin treasuries have stopped to use delegated stacking due to the global unlocking of STX tokens with the Stacks Nakamoto release.

The CCIP-020[^2] has disabled stacking and mining City coin tokens. However, the future of the treasury has not been finalized. There is a strong opinion in the community to initiate redeemption for the treasuries of MIA and NYC tokens. The details will be described in CCIP-022[^3]. It is expected that NYC coin treasury will be ready for redemption first.

Based on the uncertainty of the details in CCIP-022, this CCIP intends to:

- update .pox-3 to .pox-4 for delegated stacking in MIA coin treasury contract in continuation of the previous delegated stacking process.

The same proposal can be executed later for NYC coin treasury if demanded.

## Specification

### Treasury Contracts

The current treasury contracts used with mining directly call the `.pox-3` contract, which was replaced with `.pox-4` in the Stacks 2.5 release[^1].

CCIP-019 will implement a new version of the treasury contract `ccd002-treasury-v3` that replaces `.pox-3` with `.pox-4`, as well as move the balances and perform the delegated stacking.

The pool operator send stacking rewards to a new reward-treasury contract. The community will define in the future how to spend the stacking rewards.

> [!NOTE]
> This only affects the mining treasuries for each city, the addresses for the stacking treasuries will remain the same

| City | Current                   | New                       |
| ---- | ------------------------- | ------------------------- |
| MIA  | ccd002-treasury-mining-v2 | ccd002-treasury-mining-v3 |
| MIA  | ccd002-treasury-stacking  | _(no change)_             |

### Proposal Voting

This proposal is a continuation of the current delegated stacking process and does not require a vote.

## Backwards Compatibility

This CCIP only changes the treasury contract. All traits etc. are unchanged. The treasury continues to receive rewards.

## Activation

The CCIP would be activate via direct execution defined in CCIP-001.

## Reference Implementations

The reference implementation is published in the protocol repo[^4].

## Footnotes

[^1]: https://stacks.org/halving-on-horizon-nakamoto
[^2]: https://github.com/citycoins/governance/blob/main/ccips/ccip-020/ccip-020-graceful-protocol-shutdown.md
[^3]: https://github.com/citycoins/governance/pull/34
[^4]: https://github.com/citycoins/protocol/pull/67
