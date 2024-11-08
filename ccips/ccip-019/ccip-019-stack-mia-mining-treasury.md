# CCIP-019

## Preamble

| CCIP Number   | 019                                              |
| ------------- | ------------------------------------------------ |
| Title         | Stack MIA Mining Treasury with PoX 4             |
| Author(s)     | Jason Schrader jason@joinfreehold.com            |
|               | Tim Butterfield tim@timbutterfield.com           |
|               | Friedger Müffke mail@friedger.de                 |
| Consideration | Governance, Economic, Technical                  |
| Type          | Standard                                         |
| Status        | Ratified                                         |
| Created       | 2024-01-04                                       |
| License       | BSD-2-Clause                                     |
| Supplements   | CCIP-006, CCIP-012, CCIP-013, CCIP-014, CCIP-020 |

## Introduction

When the Stacks Nakamoto release[^1] was implemented, a new version of the PoX contract `.pox-4` was deployed and activated. The previous versions of the PoX contract were used by the treasury contracts to perform delegated stacking of STX. At the time of the release all STX tokens were unlocked from the old PoX contract, and both CityCoin treasuries stopped their use of delegated stacking.

CCIP-020[^2] disabled mining and stacking of CityCoin tokens, and CCIP-022[^3] started the liquidation of the NYC mining treasury. However, the future of the MIA treasury has not been finalized. Until then, this CCIP will stack the MIA mining treasury using pox-4 to earn stacking rewards.

The stacking rewards will be collected in a separate treasury and used for future redemption or other purposes.

This CCIP intends to:

- implement the CCIP-015[^4] voting mechanism as part of a DAO proposal
- update .pox-3 to .pox-4 for delegated stacking in MIA mining treasury contract
- allow for ending the stacking of the treasury in order to start liquidation

## Specification

### Treasury Contracts

The current treasury contracts used with mining directly call the `.pox-3` contract, which was replaced with `.pox-4` in the Stacks 2.5 release[^1].

CCIP-019 will implement a new version of the treasury contract `ccd002-treasury-v3` that replaces `.pox-3` with `.pox-4`, as well as move the balances and perform the delegated stacking.

The pool operator send stacking rewards to a new, separate treasury contract for the rewards that also supports delegated stacking.

The accrued stacking rewards in STX are intended to be used for future liquidation of the MIA mining treasury. Any action with the stacking rewards or the mining treasury requires a separate proposal.

> [!NOTE]
> This only affects the mining treasuries for MIA, the addresses for the MIA stacking treasuries will remain the same. MIA stacking rewards and MIA mining rewards can still be claimed.

| City | Current                       | New                            |
| ---- | ----------------------------- | ------------------------------ |
| MIA  | ccd002-treasury-mia-mining-v2 | ccd002-treasury-mia-mining-v3  |
| MIA  | ccd002-treasury-mia-stacking  | _(no change)_                  |
| MIA  |                               | ccd002-treasury-mia-rewards-v3 |

The new `ccd002-treasury-mia-rewards-v3` contract will collect and manage the stacking rewards.

### Stacking Delegation

Both the mining (ccd002-treasury-mia-mining-v3) and rewards (ccd002-treasury-mia-rewards-v3) treasuries will delegate up to 50 million STX each to the `SP21YTSM60CAY6D011EZVEVNKXVW8FVZE198XEFFP.pox4-fast-pool-v3` pool.

When MIA starts liquidation, the delegated stacking can be revoked through a proposal and the STX will be unlocked.

In case of a hard fork and if a new version of the PoX contract is activated, the delegated stacking should continue with a new treasury and the new PoX contract.

## Backwards Compatibility

This CCIP adds two new treasury contracts for the MIA treasuries, and is not backwards compatible.

## Activation

This CCIP will be voted on using a vote contract that adheres to CCIP-015[^4] using the last two active stacking cycles for the protocol.

- MIA cycles 82 and 83

No scale factor is required.

Voting will begin when the contract is deployed and continue until the proposal is executed.

This CCIP follows the same voting format as CCIP-022[^5] except only MIA votes are counted.

- the votes are tallied and available in read-only functions
- the proposal will not pass unless the majority of voters vote yes and 25% of voting MIA tokens vote yes

## Reference Implementations

The reference implementation is published in the CityCoins protocol repository[^6].

## Footnotes

[^1]: https://stacks.org/halving-on-horizon-nakamoto
[^2]: https://github.com/citycoins/governance/blob/main/ccips/ccip-020/ccip-020-graceful-protocol-shutdown.md
[^3]: https://github.com/citycoins/governance/pull/34
[^4]: https://github.com/citycoins/governance/blob/main/ccips/ccip-015/ccip-015-community-proposal-voting-process.md
[^5]: https://github.com/citycoins/governance/blob/main/ccips/ccip-022/ccip-022-citycoins-treasury-redemption-nyc.md
[^6]: https://github.com/citycoins/protocol/pull/67
