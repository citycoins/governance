# CCIP-019

## Preamble

| CCIP Number   | 019                                              |
| ------------- | ------------------------------------------------ |
| Title         | MIA PoX 4 Stacking                               |
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

With the Stacks Nakomoto release[^1], a new version of the PoX contract `.pox-4` was released. The previous versions of the PoX contracts were used by the treasury contracts to perform delegated stacking (STX). Both CityCoin treasuries have stopped to use delegated stacking due to the global unlocking of STX tokens with the Stacks Nakamoto release.

The CCIP-020[^2] has disabled stacking and mining City coin tokens. The CCIP-022[^3] has started liquidation of NYC token. However, the future of the MIA treasury has not been finalized. Until then, the treasury shall be stacked using pox-4 to earn stacking rewards. The stacking rewards should be collected and used for future redemption or other purposes.

This CCIP intends to:

- update .pox-3 to .pox-4 for delegated stacking in MIA coin treasury contract in continuation of the previous delegated stacking process.
- define the end of stacking the treasury as the start of liquidation.

## Specification

### Treasury Contracts

The current treasury contracts used with mining directly call the `.pox-3` contract, which was replaced with `.pox-4` in the Stacks 2.5 release[^1].

CCIP-019 will implement a new version of the treasury contract `ccd002-treasury-v3` that replaces `.pox-3` with `.pox-4`, as well as move the balances and perform the delegated stacking.

The pool operator send stacking rewards to a new reward-treasury contract (ccd002-treasury-mia-stx-stacking-v3).

Delegated stacking is also applied to the new reward-treasury contract.

The stacking rewards in STX shall be used for future liquidation. Other use of the rewards require a new proposal in the future.

> [!NOTE]
> This only affects the mining treasuries for MIA city, the addresses for the MIA stacking treasuries will remain the same. MIA stacking rewards and MIA mining rewards can still be claimed. 

| City | Current                   | New                             |
| ---- | ------------------------- | ------------------------------- |
| MIA  | ccd002-treasury-mining-v2 | ccd002-treasury-mining-v3       |
| MIA  | ccd002-treasury-stacking  | _(no change)_                   |
| MIA  |                           | ccd002-treasury-stx-stacking-v3 |

### End of Delegated Stacking

Delgated stacking shall be revoked as soon as liquidation similar to CIP-022 but for MIA tokens starts. 

In case of a hard fork and if a new version of the PoX contract is activated, the delegated stacking should continue with the new PoX contract.

## Backwards Compatibility

This CCIP only changes the treasury mining contract and creates a new stacking reward treasury contract. It is not backwards compatible.

## Activation
This CCIP will be voted on using a vote contract that adheres to CCIP-015[^4] using the last two active stacking cycles for the protocol.

- MIA cycles 82 and 83
- NYC cycles 82 and 83

No scale factor is required.

Voting will begin when the contract is deployed and continue until the proposal is executed.

This CCIP follows the same voting format as CCIP-022[^5] except only MIA votes are counted.

- the votes are tallied and available in read-only functions
- the proposal will not pass unless 25% of MIA have participated

## Reference Implementations

The reference implementation is published in the protocol repo[^6].

## Footnotes

[^1]: https://stacks.org/halving-on-horizon-nakamoto
[^2]: https://github.com/citycoins/governance/blob/main/ccips/ccip-020/ccip-020-graceful-protocol-shutdown.md
[^3]: https://github.com/citycoins/governance/pull/34
[^4]: https://github.com/citycoins/governance/blob/main/ccips/ccip-015/ccip-015-community-proposal-voting-process.md
[^5]: https://github.com/citycoins/governance/blob/main/ccips/ccip-022/ccip-022-citycoins-treasury-redemption-nyc.md
[^6]: https://github.com/citycoins/protocol/pull/67
