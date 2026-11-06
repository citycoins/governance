# CCIP-027

## Preamble

| CCIP Number   | 027                                |
| ------------- | ---------------------------------- |
| Title         | MiamiCoin PoX-5 Staking            |
| Author(s)     | Friedger Müffke mail@friedger.de   |
| Consideration | Governance, Technical, Economic    |
| Type          | Standard                           |
| Status        | Draft                              |
| Created       | 2026-08-05                         |
| License       | BSD-2-Clause                       |
| Supplements   | CCIP-026                           |

## Introduction

The MiamiCoin mining treasury stopped earning at the Stacks 4.0 hard fork.

Until then, the treasury (`ccd002-treasury-mia-mining-v3`) was delegated to Fast Pool under `pox-4`, and the resulting stacking yield was paid into the rewards treasury (`ccd002-treasury-mia-rewards-v3`), which is the account CCIP-026 pays burn-to-exit redeemers from. Stacks 4.0 replaced `pox-4` with `pox-5`, and `pox-5` has no delegation: `stake` locks the STX of the calling principal, and there is no equivalent of `delegate-stx` / `delegate-stack-stx`. The treasury's delegation is therefore inert. As of burn block 961,218 the mining treasury holds 10,241,497.066794 STX with **zero locked**.

This CCIP proposes a new DAO extension that can hold the treasury's STX and call `pox-5` directly, with rewards paid in sBTC.

## Specification

### The extension

`ccd002-treasury-mia-mining-v3` cannot stake under PoX-5 because the function with pox-4 will fail.

`ccd014-pox5-staking-mia` is a new DAO extension that can hold the mining treasury's STX, can call PoX-5 directly, and can forwards the sBTC rewards on. Its properties, in order of importance:

1. **Both destinations are constants, not parameters.** STX can only ever go back to `ccd002-treasury-mia-mining-v3` (`return-stx`), and sBTC rewards can only ever go to `ccd002-treasury-mia-rewards-v3` (`forward-rewards`), the account `ccd013-burn-to-exit-mia` pays redeemers from. No proposal, and no caller, can redirect either.

2. **Upkeep is permissionless, exit is governed.** `stake-stx`, `stake-update` and `forward-rewards` may be called by anyone: the worst a caller can do is keep the position alive or move rewards to their only legal destination sooner. `unstake-stx` and `return-stx` are DAO-gated, because leaving the signer set and withdrawing the principal are governance decisions rather than routine maintenance.

   The corollary is that leaving takes two steps in one transaction. `unstake-stx` alone only shortens the lock to the end of the current reward cycle, and `stake-update` — permissionless — re-locks it for another `num-cycles` cycles, so an exit proposal must also `set-num-cycles` to `u0`. At zero, `stake-update` is refused by the extension itself with `ERR_INVALID_NUM_CYCLES` and `stake-stx` reaches PoX-5 with a zero-cycle request and is refused there, so the balance unlocks on schedule with nothing able to renew it.

3. **One approved signer.** Trait arguments cannot be stored in Clarity, so callers pass a `signer-manager-trait` implementation and the contract checks it against a DAO-set `approved-signer` allowlist of exactly one principal. Clearing it blocks new stakes without disturbing an existing position. `set-num-cycles` fixes the lock length `stake-stx` uses and caps what a single `stake-update` may add — PoX-5 caps the resulting *total* at 96 cycles rather than the increment, so the extension derives the increment from the live position (`get-cycles-to-extend`) and takes whichever is smaller. A position already locked for 96 cycles therefore has no headroom until a cycle elapses, and `stake-update` returns `ERR_INVALID_NUM_CYCLES` until it does.

4. **Rewards in sBTC rather than Bitcoin.** `stake` passes `signer-calldata` of `none`. In the canonical signer-manager implementations, calldata carrying a pox-addr routes a staker's rewards to an L1 Bitcoin address through an sBTC withdrawal request; omitting it pays sBTC directly to the staker. `none` is the parameter that selects sBTC.

The cost of this shape is that the treasury is split in two: `ccd002-treasury-mia-mining-v3` keeps the treasury role and the DAO's tooling, while the assets sit in a contract that is not a `ccd002` treasury and has no deposit, withdraw or allow-list surface of its own. That is deliberate — the extension's whole attack surface is two constant destinations — and `return-stx` puts the balance back under the treasury whenever the DAO wants it there.

### The proposal

`ccip-027-mia-pox5-staking`, executed through `ccd001-direct-execute`:

1. Enables `ccd014-pox5-staking-mia` in `base-dao`.
2. Restricts it to a single signer-manager, Fast Pool max500.
3. Sets the lock length to 96 cycles, PoX-5's maximum.
4. Allows `sbtc-token` in `ccd002-treasury-mia-rewards-v3`, so forwarded rewards can be withdrawn later.
5. Moves the mining treasury's full STX balance into the extension, guarded by a 10,000,000 STX sanity floor read live from the treasury.

**Timing.** Staking is not performed here. `pox-5.stake` reverts during a prepare phase, so a proposal that staked inside `execute` would force the approvers to land the third signal outside a prepare window. Instead `stake-stx` is permissionless and called separately once the proposal has executed and the chain is outside the prepare window, which keeps execution timing independent of approver availability. Between the two the STX sits unstaked in the extension, where the only thing anyone can do with it is stake it with the approved signer.

### Signer selection

`signer-guide.fastpool.org` lists 35 registered signers for cycle 140. Filtering to those that are registered, open to anyone, and running a canonical implementation leaves 22 candidates, of which 21 charge 0 bips.

This CCIP proposes **`SPMPMA1V6P430M8C91QS1G9XJ95S59JS1TZFZ4Q4.fastpool-max500-signer-manager`**:

| Property        | Value                                                                |
| --------------- | -------------------------------------------------------------------- |
| Fee             | 0 bips, with a hard 5% (500 bips) ceiling written into the contract    |
| Implementation  | v2 of the reference signer-manager, canonical source hash              |
| Open to anyone  | Yes                                                                   |
| Registered      | Yes, with an active signer key grant                                  |
| sBTC rewards    | Yes, when no pox-addr calldata is supplied                             |

The 500 bips ceiling is the reason to prefer it over the 0-bip managers that have no cap: an admin there can raise the fee to anything, where max500 cannot exceed 5% and must queue increases two cycles ahead (decreases take effect immediately). v2 also splits reward settlement from payout, so a small reward can accumulate across cycles instead of being spent on Bitcoin fees.

The operator is Fast Pool, which ran the `pox4-fast-pool-v3` delegation the treasury is still inertly pointed at. Choosing it keeps operational continuity across the fork. The choice is not load-bearing, though migrating is not quite free either: `set-approved-signer` is one proposal, and the migration itself rides on `stake-update`, which needs headroom under PoX-5's 96-cycle cap. At the configured length the position has none until a cycle elapses, so a migration lands within one cycle of the proposal — or immediately, at the cost of an exit proposal and a fresh `stake-stx` a cycle later.

Signer risk is bounded but real. A signer that fails to sign blocks earns no rewards for its stakers, and the signer-manager contract is the party that calls `pox-5.claim-rewards` and holds sBTC between that call and each staker's `claim-staker-rewards`. It cannot touch staked STX, which PoX-5 locks against the staker principal.

### Amount and duration

The full mining treasury balance, 10,241,497.066794 STX, is staked. This is idle capital: CCIP-026's redemption ratio was snapshotted into a data var at initialization and reads from storage, not from the live treasury balance, so moving or staking this STX does not change any redeemer's payout. `ccd013-burn-to-exit-mia` pays exclusively from the rewards treasury and never reads the mining treasury again.

For scale, 10.24M STX is roughly 2.3% of the 446,974,094 STX stacked in cycle 140, and comfortably above PoX-5's 50,000 STX signer-set minimum.

The lock length is set to 96 cycles, PoX-5's maximum — roughly 3.85 years, far longer than `ccd001-direct-execute`'s remaining ~176 days of runway, and longer than PoX-5 itself is expected to last unchanged; a hard fork inside that window will re-open the question regardless. It is a default rather than a commitment: `pox-5.unstake` shortens any position to the end of the current reward cycle, so the DAO's exit latency is one cycle regardless of the configured length. Staking for the maximum simply means nobody has to call `stake-update` to keep the position alive.

## Related Work

CCIP-026 established burn-to-exit for MIA and is the reason this proposal matters: it created a redemption path whose only funding source was stacking yield. CCIP-021 and the pox-4 delegation proposals established the pattern of the DAO delegating treasury STX for yield; this CCIP is that pattern rebuilt for a protocol that no longer supports delegation.

An alternative direction, converting the treasury to sBTC and participating through PoX-5 bonds rather than STX staking, is discussed in citycoins/governance issue #52. That is a larger change with market-price exposure (a bond requires maintaining a STX component alongside the sBTC). This CCIP restores income without changing the treasury's asset composition, and does not preclude a later move to bonds.

## Backwards Compatibility

None required. `ccd013-burn-to-exit-mia`, the token contracts, and the ratio snapshot are untouched. The stale `pox-4` delegation on the mining treasury is left in place; it locks nothing once the balance has moved, and revoking it is optional cleanup that can ride along with a later proposal.

## The yield is sBTC

`ccd013-burn-to-exit-mia.redeem-mia` pays STX, drawn from the rewards treasury. Under this CCIP the rewards treasury accumulates **sBTC**, so CCIP-027 on its own restores income without restarting STX redemptions.

That mismatch is resolved by the companion proposal to redeem MIA for sBTC and burn it. Once redemption is denominated in sBTC, the asset this proposal delivers and the asset holders exit into are the same, and no conversion step sits between them. Routing PoX-5 rewards into `ccd002-treasury-mia-rewards-v3` as sBTC puts them in the account that redemption pays from.

## Activation

The CCIP-027 continues the yield process established in the past therefore it is executed through `ccd001-direct-execute`, which remains enabled, requires 3 approver signals, and does not sunset until tenure height 277,428 (roughly 25,400 tenures, ~6 months, from the time of writing).

Post-execution steps, both permissionless:

1. Call `stake-stx` outside a prepare phase. The position begins at the following reward cycle.
2. Once rewards accrue, call `claim-rewards` on the signer-manager for the cycle, then `claim-staker-rewards` for the extension, then `forward-rewards` on the extension. All three are permissionless and none of them is anybody's obligation: rewards sit in the signer-manager until somebody runs the chain, so the DAO should name who does, or accept that the sBTC simply accumulates until it does.

## Reference Implementations

- `contracts/ccd014-pox5-staking-mia.clar`
- `contracts/ccip-027-mia-pox5-staking.clar`

Both must be deployed from the same address for the proposal's `.contract-name` references to resolve.

See https://github.com/friedger/ccip27-pox5