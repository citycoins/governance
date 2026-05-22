# CCIP-026

## Preamble

| CCIP Number   | 026                                   |
| ------------- | ------------------------------------- |
| Title         | MiamiCoin Burn to Exit                |
| Author(s)     | Jason Schrader jason@joinfreehold.com |
|               | Friedger Müffke mail@friedger.de      |
|               | hanjumeok wnsgur5144@gmail.com        |
| Consideration | Governance, Economic                  |
| Type          | Standard                              |
| Status        | Ratified                              |
| Created       | 2025-02-17                            |
| Ratified      | 2026-05-15                            |
| License       | BSD-2-Clause                          |
| Supplements   | CCIP-020, CCIP-024                    |

## Introduction

Following the MiamiCoin community signal vote[^1] and the graceful protocol shutdown[^2], and in absence of any official response to the community's expressed desires, this CCIP proposes implementing a burn-to-exit mechanism for MIA holders using the secondary treasury[^3]. This mechanism will provide MIA holders with an optional path to burn their tokens in exchange for STX at a ratio determined by the treasury balance and total supply at initialization, while leaving the original city treasury[^4] untouched. Token holders may choose to participate in the burn-to-exit or maintain their position in case of future developments regarding the main treasury.

## Specification

### Burn-to-Exit Mechanism

The implementation will create a new extension that:

1. will call `revoke-delegate-stx` on the rewards treasury to make the STX liquid.

2. can access STX in the secondary treasury (ccd002-treasury-mia-rewards-v3) while enabled.

   - The extension only has access to the rewards treasury by design - it is a separate contract from the original city treasury.
   - The burn-to-exit mechanism can be disabled through a separate proposal.

3. calculates the redemption ratio dynamically at initialization, using the same treasury-to-total-supply calculation used in CCIP-022 for NYC.

   - Ratio = Mining Treasury Balance / Total MIA Supply
   - The ratio represents each MIA holder's proportional share as if the mining treasury were being liquidated.
   - The ratio is calculated once when redemptions are initialized and remains fixed thereafter.
   - The actual ratio is locked when `direct-execute` reaches threshold and triggers `execute → initialize-redemption`. Values below are estimates taken at drafting time across cycles 126 and 127; the mining treasury balance has been stable since shutdown, but the final values are whatever the chain shows at execution.
   - Example estimate (cycle 126: Bitcoin blocks 930,650 → 932,749 / Stacks blocks 5,655,770 → 5,946,919; cycle 127: Bitcoin blocks 932,750 → 934,849 / Stacks blocks 5,947,310 → 6,294,407):
     - Mining Treasury Balance (locked + unlocked via `stx-account`): ~10,241,497 STX
     - Total MIA Supply: ~5,988,905,152 MIA (V1 + V2)
     - This yields approximately 1,710 STX per 1M MIA

4. is designed to run until the treasury is empty.

5. Functionality:

   - Any MIA holder can participate by calling `redeem-mia` with the amount of MIA (in micro-MIA) they wish to redeem.
   - Claims are limited to 10,000,000 MIA per transaction. If a user attempts to redeem more, the transaction will process up to 10,000,000 MIA, and the remainder of the user's balance will be unaffected.
   - Users can call the redemption function multiple times to redeem their full balance in batches.
   - Both MIA V1 and V2 tokens are supported. V1 tokens are redeemed first, then V2 if the requested amount exceeds V1 balance.
   - MIA tokens will be burned upon redemption.
   - STX will be transferred from the rewards treasury to the participant's wallet.
   - The contract tracks cumulative redemptions per address (total MIA burned and STX received).

### Technical Implementation

| Contract                       | Purpose            | Access                 |
| ------------------------------ | ------------------ | ---------------------- |
| ccd002-treasury-mia-rewards-v3 | Secondary Treasury | Burn-to-Exit Extension |
| ccd002-treasury-mia-mining-v3  | Main City Treasury | No Access              |

The extension will only have access to the MIA rewards treasury (ccd002-treasury-mia-rewards-v3) and will:

- Accept MIA tokens from users (V1 and V2)
- Calculate the STX return based on the redemption ratio
- Burn the received MIA tokens
- Transfer the calculated STX to the user via `withdraw-stx` on the treasury
- Track total MIA burned and STX distributed
- Include read-only functions to check available STX balance
- Include read-only functions to check addresses and redemption history
- Cap redemption amount to remaining treasury balance if it would exceed available STX

> [!NOTE]
> The burn-to-exit mechanism is one-way and irreversible. Once MIA tokens are burned, they cannot be recovered.

### Treasury Management

The two treasuries play different roles in the burn-to-exit design — one sizes the redemption ratio, the other funds the payouts:

- **Mining treasury** (`ccd002-treasury-mia-mining-v3`[^4]) — the snapshot source for the redemption ratio. The extension reads the locked + unlocked sum via `stx-account` and stores it in `mining-treasury-ustx` at `initialize-redemption`. No STX is transferred out of this treasury by the burn-to-exit extension; it continues stacking and emits yield to the rewards treasury over time. Balance is ~10,241,497 STX (stable since shutdown).

- **Rewards treasury** (`ccd002-treasury-mia-rewards-v3`[^3]) — the payout source. Each `redeem-mia` call pulls STX from this treasury via `withdraw-stx`. At drafting time (cycle 126) the rewards treasury held ~1,217,764 STX; it has since grown with stacking yield to ~1,533,479 STX total (of which ~18,475 STX is currently unlocked and immediately spendable; the rest unlocks as the underlying stacking cycles complete). The rewards treasury receives stacking payouts from two sources (per CCIP-012[^7]/CCIP-013[^8]):
  - Its own stacked STX (compounding)
  - The mining treasury's stacking payouts

Because the snapshot ratio is sized against the full mining treasury (~10.24M STX) but payouts come from the rewards treasury (which fills gradually), the contract enforces a per-redemption cap: if a calculated STX payout would exceed the rewards treasury's currently spendable balance, the contract reduces both the STX payout AND the MIA burn amount proportionally, so users only burn for STX they actually receive. The unused portion of the user's intended redemption can be claimed in a later transaction once more STX has unlocked.

- Upon activation, `revoke-delegate-stx` is called on the **rewards** treasury (not the mining treasury) to make the STX held there liquid for redemptions.
- The rewards treasury continues to be refilled by stacking payouts as long as the mining treasury remains stacked.
- The extension maintains a public record of all redemptions (cumulative MIA burned and STX received, per address and in aggregate).
- The burn-to-exit mechanism remains enabled until the rewards treasury is drained (or disabled via a separate proposal).
- Participation is optional, allowing holders to maintain positions if desired.

## Backwards Compatibility

This CCIP supplements CCIP-020 and CCIP-024 by introducing a new, independent mechanism that does not alter the contracts or functionality established by the supplemented CCIPs.

## Activation

### Voting Method

This CCIP uses an on-chain Merkle-snapshot voting method rather than the `at-block`-based snapshot used in CCIP-015[^9]. The `at-block` keyword was removed in the Stacks Nakamoto release (Epoch 3), so historical balance reads via `at-block` are no longer possible inside Clarity. CCIP-026 commits a Merkle root of the eligible-voter set on-chain and verifies each ballot with a Merkle proof supplied by the voter's client.

**Snapshot source.** Voter eligibility and weight come from MIA stacking data in the last two active MIA stacking cycles before shutdown:

- MIA cycle 82 (starts at Stacks block 145,643 / Bitcoin block 838,250)
- MIA cycle 83 (starts at Stacks block 147,282 / Bitcoin block 840,350)

For each stacker, `weight = (cycle82Stacked + cycle83Stacked) / 2`, scaled by `VOTE_SCALE_FACTOR = 10^16`. The 299 non-zero entries are sorted alphabetically by address and padded to 512 leaves (Merkle depth 9), producing a 32-byte root committed in the proposal contract:

```
snapshot-merkle-root = 0x776695e7e2659b4a92ed54d411456f244568e2572d8e8133fc0c2381c9d154b3
```

The build is fully deterministic and independently reproducible from on-chain stacking data; both the contract's verification logic and the snapshot builder use tagged SHA-256 (`merkle-leaf` and `merkle-parent` domain separators) and SIP-005 consensus-buff encoding to ensure the same inputs always yield the same root. A verification page is available at the FastPool reference UI[^10], and the snapshot can be regenerated from the `simulations/calculate-mia-votes.ts` script in the contract repository[^11].

**Vote mechanics.** Only addresses present in the snapshot can vote, with weight fixed to their cycle-82/83 average — not current balance. The voter submits `(vote bool, scaled-mia-vote-amount uint, proof (list 9 (buff 32)), positions (list 9 bool))`; the contract recomputes the leaf, verifies the Merkle path against the hardcoded root, and records the ballot. Re-voting in the same direction returns `ERR_VOTED_ALREADY`; flipping direction adjusts the tally without re-verifying the proof. Double-vote prevention is enforced via `voter-id` keying.

**Window.** Voting opens when the proposal contract is deployed (`vote-start = burn-block-height` at deploy) and runs for `VOTE_LENGTH = 2,016` Bitcoin blocks (~2 weeks).

### Voting Outcome

The proposal contract was deployed on mainnet at Stacks block 7,754,807 / Bitcoin block 946,772 (2026-04-26) by `SP2PABAF9FTAJYNFZH93XENAJ8FVY99RRM50D2JG9` (friedgerpool.id)[^5]. The deployment matches the citycoins source repository[^11] byte-for-byte.

Voting ran from Bitcoin block 946,772 to 948,788 (cycles 133–134) and closed with:

- **Yes:** 1,638,177,567 MIA across 39 voters
- **No:** 0 MIA across 0 voters
- **Turnout:** 61.67% of 2,656,102,117 MIA total eligible voting power
- **Threshold:** >50% yes vs no (met)

`is-executable` returns `(ok true)`; `is-vote-active` returns `(some false)`.

### Ratification & Execution

The proposal cleared the `ccd001-direct-execute` 3-of-5 multisig over the approver set established by CCIP-012. Three signals were collected:

| # | Signer | Tx | Bitcoin block | Date (UTC) |
| - | ------ | -- | -------------:| ---- |
| 1 | friedger.btc (`SPN4Y…8C3X`) | `0x1f1b9c22d60b4690e21dceaafb83661f0ae40cdf929a52ae433b35fd4e4e99bf` | 948,802 | 2026-05-10 |
| 2 | `SP3YY…HJQ` | `0x04cd49c2f08ddb5b74ead277995260b1d451e9da184be2a63180165fe360aa80` | 949,528 | 2026-05-15 |
| 3 | `SP7DG…JDE` | `0x5d9e291f55b59df50608c64fe428e1d0201be4cb760b7db0004e752ea1c6810d` | 949,551 | 2026-05-15 |

The third signal completed the threshold and triggered `ccd001-direct-execute` to call `ccip026-miamicoin-burn-to-exit.execute`, which atomically:

1. Enabled the `ccd013-burn-to-exit-mia` extension in the DAO
2. Called `initialize-redemption`, which:
   - Revoked the rewards treasury's stacking delegation (`revoke-delegate-stx`)
   - Snapshotted `mining-treasury-ustx` from the mining treasury's locked + unlocked balance via `stx-account`
   - Computed and stored the redemption ratio (`mining-treasury-ustx * 10^6 / total-MIA-supply`)
   - Set `redemptions-enabled = true`

**Locked redemption ratio:** `1,710` (µSTX-per-µMIA × 10⁶), equivalent to **1,710 STX per 1,000,000 MIA**.

Redemptions opened at Stacks block 7,961,948 / Bitcoin block 949,551 (2026-05-15) and are running. As of 2026-05-21, ~906.5M MIA has been burned and ~1,550,074 STX has been paid out across the redemption window; the rewards treasury continues to refill from stacking yield on the mining treasury until the latter is drained.

## Reference Implementations

The contracts are deployed on Stacks mainnet at block 7,754,807 (Bitcoin block 946,772) by `SP2PABAF9FTAJYNFZH93XENAJ8FVY99RRM50D2JG9`. The deployed source matches the citycoins reference repository[^11] byte-for-byte (modulo a trailing newline). An independent diff against the deployed bytecode confirmed no divergence in semantic content.

- CCIP-026 Proposal: `ccip026-miamicoin-burn-to-exit`[^5]
- CCD013 Burn Extension: `ccd013-burn-to-exit-mia`[^6]
- DAO multisig (execution): `ccd001-direct-execute`[^12]
- Vote verification UI: FastPool reference site[^10]

## Footnotes

[^1]: https://github.com/citycoins/governance/blob/main/ccips/ccip-024/ccip-024-miamicoin-community-signal-vote.md
[^2]: https://github.com/citycoins/governance/blob/main/ccips/ccip-020/ccip-020-graceful-protocol-shutdown.md
[^3]: https://explorer.hiro.so/txid/SP8A9HZ3PKST0S42VM9523Z9NV42SZ026V4K39WH.ccd002-treasury-mia-rewards-v3?chain=mainnet
[^4]: https://explorer.hiro.so/txid/SP8A9HZ3PKST0S42VM9523Z9NV42SZ026V4K39WH.ccd002-treasury-mia-mining-v3?chain=mainnet
[^5]: https://explorer.hiro.so/txid/SP2PABAF9FTAJYNFZH93XENAJ8FVY99RRM50D2JG9.ccip026-miamicoin-burn-to-exit?chain=mainnet
[^6]: https://explorer.hiro.so/txid/SP2PABAF9FTAJYNFZH93XENAJ8FVY99RRM50D2JG9.ccd013-burn-to-exit-mia?chain=mainnet
[^7]: https://github.com/citycoins/governance/blob/main/ccips/ccip-012/ccip-012-stabilize-emissions-and-treasuries.md
[^8]: https://github.com/citycoins/governance/blob/main/ccips/ccip-013/ccip-013-stabilize-protocol-and-simplify-contracts.md
[^9]: https://github.com/citycoins/governance/blob/main/ccips/ccip-015/ccip-015-community-proposal-voting-process.md
[^10]: https://mia-burn-to-exit.fastpool.org/#/verify
[^11]: https://github.com/citycoins/clarity-ccip-026
[^12]: https://explorer.hiro.so/txid/SP8A9HZ3PKST0S42VM9523Z9NV42SZ026V4K39WH.ccd001-direct-execute?chain=mainnet
