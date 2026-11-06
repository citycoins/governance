# CCIP-028

## Preamble

| CCIP Number   | 028                                            |
| ------------- | ---------------------------------------------- |
| Title         | MiamiCoin Fair Redemption with sBTC Rewards    |
| Author(s)     | Rapha (github.com/Rapha-btc)                   |
| Consideration | Governance, Economic, Technical                |
| Type          | Standard                                       |
| Status        | Draft                                          |
| Created       | 2026-08-13                                     |
| License       | BSD-2-Clause                                   |
| Supplements   | CCIP-026, CCIP-027                             |

## Introduction

CCIP-027 re-entered the MiamiCoin treasury into Proof of Transfer under PoX-5.
Rewards now arrive as sBTC, while the burn-to-exit mechanism established by
CCIP-026 pays in STX. Until those two are connected, sBTC accumulates in the
rewards treasury and no MIA is retired.

This CCIP proposes to convert the sBTC rewards to STX through a keeperless swap
vault, route that STX only into a fair order book that lets holders name their
own exit price, and burn every MIA the book acquires.

How a holder exits stays the same. Offers are posted at or below par, the
cheapest offers fill first, offers may be cancelled at any time, and sellers are
paid in STX.

Three things change:

1. Par is recomputed from the current treasury and supply, which accounts for
   the MIA already burned.
2. The DAO is the only buyer. All MIA the book acquires is burned, so there is
   no settler and no retained spread.
3. The book is a new contract. Resting offers on the existing book do not
   migrate automatically.

## Problem Statement

### The rewards cannot reach the redemption mechanism

CCIP-027 moved 10,241,497.066794 STX from `ccd002-treasury-mia-mining-v3` into
`ccd014-pox5-staking-mia` and staked it from reward cycle 141 for 96 cycles,
unlocking at cycle 237. The treasury is intact and earning.

The reward path is already permissionless:

```
pox-5 -> ccd014-pox5-staking-mia -> forward-rewards -> ccd002-treasury-mia-rewards-v3
```

`forward-rewards` takes no arguments, may be called by anyone, and sweeps the
full sBTC balance. CCIP-027 additionally called `set-allowed` for `sbtc-token` on
the rewards treasury, with the stated intent that a future proposal would
withdraw it. This is that proposal.

What is missing is the conversion from sBTC to the STX the redemption ratio is
denominated in.

### The redemption ratio is stale

`ccd013-burn-to-exit-mia` pays 1710, that is 1,710 STX per 1,000,000 MIA. That
was correct when it was set. Every exit round since has burned MIA, shrinking
supply while the treasury is untouched, so the true ratio has risen and holders
redeeming today receive less than their claim is worth.

Measured on-chain on 2026-08-13:

| Input                          | Value                     |
| ------------------------------ | ------------------------- |
| MIA v1 supply                  | 260,130,901 MIA           |
| MIA v2 supply                  | 4,754,074,853.491373 MIA  |
| Combined supply                | 5,014,205,754 MIA         |
| Treasury, now in ccd014        | 10,241,497.066794 STX     |
| Ratio by the DAO's formula     | 2042                      |
| Ratio currently paid           | 1710                      |

The formula gives 19.4% more than holders are currently paid. The figure moves
with every burn, so what should be ratified is the formula, not the number above.
A mainnet-fork rehearsal of the activation computed 2,042.49 STX per 1,000,000
MIA.

The ratio cannot be corrected in place. `ccd013.initialize-redemption` sets
`redemptions-enabled` permanently and the contract exposes no setter, so 1710 is
final there. Its treasury reader also still points at
`ccd002-treasury-mia-mining-v3`, which CCIP-027 emptied into
`ccd014-pox5-staking-mia`. It reads 0 STX and would revert with
`ERR_GETTING_REDEMPTION_BALANCE`. A corrected par therefore needs a new
contract, not a configuration change.

## Specification

### Overview

```
  pox-5 rewards (sBTC)
          |
          v
  ccd014-pox5-staking-mia
          |  forward-rewards           (permissionless, already deployed)
          v
  ccd002-treasury-mia-rewards-v3
          |  withdraw-ft via fund-from-treasury   (authorised by this proposal)
          v
  ccd016-swap-vault-mia-v2             (new)
          |  1. patience:    jing-place     resting ask at oracle mid + 1 bp
          |  2. liquidation: jing-reclaim, router-swap   at oracle floor
          |  fuel-fair-book                 STX only to ccd015
          v
  ccd015-redemption-book-mia-stx       (new)
          |  cross-book: pay sellers in STX, cheapest first, at or below par
          v
      MIA v2 burned
```

Two new contracts and one activation proposal are introduced. The book and the
vault are deployed from the same address, book first, because the vault's STX
destination is bound to the book by a relative contract reference.

### 1. Par: formula-locked snapshot, refreshed by proposal

The book `ccd015-redemption-book-mia-stx` stores par as a snapshot of the DAO's
formula:

```
par = (mining-v3 locked + unlocked + ccd014 locked + unlocked) STX
      / (MIA v1 supply * 10^6 + MIA v2 supply)
```

The treasury is read with `stx-account`, so stacked (locked) STX is counted.
`update-par` takes no arguments and can only be called by the DAO or an enabled
extension. It writes the formula's current output. A proposal therefore decides
**when** par moves, never **what** it moves to.

Par must stay fixed while offers are resting, or sellers cannot price against it.
It should not stay fixed forever, because each round of burns raises what the
remaining holders are owed. This proposal sets par once at activation. Each later
refresh is a separate `ccd001-direct-execute` proposal that calls `update-par`,
ideally between rounds when the book has been crossed. Offers are checked
against par both when they are placed and when the book is crossed, so a
refresh cannot leave the book filling above par.

There is no fixed-ratio redemption in this design. A holder exits by selling into
the book at or below par. A holder who wants exactly par posts at par and waits
for rewards to reach their position.

### 2. Swap vault (sBTC -> STX)

`ccd016-swap-vault-mia-v2` is a DAO extension that holds sBTC and converts it to
STX without a keeper or an operator key. The price never comes from a caller. It
comes from a signed Pyth Lazer update that the Jing market verifies, and the mid
must agree with the on-chain DIA STX/BTC rate within a configurable band. A
caller can choose only when to act.

Required properties, as implemented:

- **Permissionless funding, no arguments.** `fund-from-treasury` pulls the
  rewards treasury's whole sBTC balance. The budget is the balance.
- **Hard-coded destinations.** STX leaves only through `fuel-fair-book`, to the
  book. sBTC leaves only through `dao-recall-sbtc`, back to the rewards
  treasury, and only by proposal.
- **A batch clock.** Funding an empty vault opens a window of 288 Bitcoin blocks
  (about two days, at most one week). Funds that arrive while the window is open
  join the batch without resetting the clock.

Each batch is sold in two phases.

1. **Patience (window open).** Anyone may call `jing-place`, which rests the
   vault's entire sBTC balance on the Jing sBTC/STX order book as a pegged ask
   at the oracle mid + 1 bp. The ask is turned off whenever the mid falls more
   than the leeway (10%) below the mid at placement, and turns back on when it
   recovers. The vault never sells to existing orders during this phase, so
   nobody can sell it below the mid while the window is open.
2. **Liquidation (window elapsed).** Anyone may call `jing-reclaim` to recover
   unsold sBTC, then `router-swap` to sell it through the Jing smart router.
   The router can split across the Jing book, Bitflow DLMM, Bitflow XYK and
   Velar. Every leg must clear above the floor `mid * (1 - slippage)`
   (slippage 1%). Each call sells at most one chunk (0.01 sBTC), with at least
   one Bitcoin block between sales. Whatever no venue takes above the floor
   stays in the vault.

**Front-running exposure.** The maker phase fills at the oracle mid and cannot
be sandwiched. The liquidation phase does touch AMM pools. The oracle floor caps
what a sandwich can extract at the slippage times the chunk, and an attacker pays
the venue fee twice. With a 1% floor, Bitflow XYK and DLMM (50 bps per side) sit
at break-even. Velar (30 bps per side) leaves at most 0.4% of a 0.01 sBTC chunk
per call. The one-block cooldown stops an attacker from chaining chunks within a
block. These bounds depend on the chunk size, the slippage setting and venue
fees, so the parameters should be revisited if any of those change.

**DAO-only tools.** By proposal, the DAO can move the floor (`jing-refloor`),
take against the Jing book at the floor (`jing-take`), run manual or emergency
router splits (`router-swap-split`, `router-swap-split-dia`), reclaim or recall
funds at any time, and change any parameter within hard-coded bounds.

### 3. Fair order book, all MIA burned

`ccd015-redemption-book-mia-stx` is an STX-denominated fork of the production
book `SPV9K21TBFAK4KNRJXF5DFP8N7W46G4V9RCJDC22.mia-fair-faktory-v2`.

Kept unchanged: the sorted offer book, insertion-sort placement, partial fills,
cheapest-first settlement, and cancellation at any time.

Changed:

- **The contract is the only buyer.** `cross-book` is permissionless, takes no
  arguments, and spends only the STX the contract already holds. There is no
  settler leg and no surplus to capture.
- **All acquired MIA is burned** in the same transaction.
- **Asks are in uSTX**, so the par check is a direct comparison against the
  stored snapshot, with no oracle.

Book limits: at most 50 offers, one offer per address, a minimum of 100,000 MIA
(set at activation) and a maximum of 10,000,000 MIA per offer. When the book is
full, a new offer is accepted only at a strictly better price, and the worst
offer is refunded. Only MIA v2 is accepted. Par counts v1 supply as well, so v1
holders must convert to v2 before they can offer.

The book has no STX withdrawal function. STX that cannot be matched against
offers at or below par stays in the book until eligible offers arrive.

The accounting consequence of burning is that a fill retires a claim worth
`par * amount` on the treasury while paying only the seller's ask. Payment comes
from staking yield and never debits the STX treasury, so the treasury `T` is
unchanged and only supply `S` shrinks:

```
par_before = T / S           par_after = T / (S - burned)
```

The full retired claim therefore accrues to holders who did not sell. The
discount does not create that surplus. It determines how much MIA is retired per
STX of yield, which is why the book fills cheapest first.

### 4. Related Work

The order book mechanism has been running in production against
`ccd013-burn-to-exit-mia` since cycle 138. Lifetime figures as of 2026-08-13:

| Metric                       | Value             |
| ---------------------------- | ----------------- |
| MIA cleared out of the queue | 42,903,974.60 MIA |
| STX paid to sellers          | 62,167.14 STX     |
| Offers currently resting     | 50                |
| MIA currently on the book    | 49,769,714.87 MIA |

The most recent settlement, on 2026-08-11, cleared 16,239,028 MIA in a single
transaction, paid three sellers at exactly the prices they had posted, and left
the redemption treasury at zero. The settler took no profit, being capped at par
by construction.

That implementation lets a settler buy and retains the spread. Section 3 replaces
the settler with the DAO and burns everything.

## Disclosure

The swap vault trades on the Jing sBTC/STX market and smart router
(`SPV9K21TBFAK4KNRJXF5DFP8N7W46G4V9RCJDC22.markets-sbtc-stx-jing-v6-4` and
`.swap-router-sbtc-stx-jing-v5-5`), deployed from the same principal as the
production fair book. _[Author to confirm: the author's role in building and
operating Jing, any grant funding, and any fees or rebates the author receives
from volume routed through it.]_ CityCoins volume routed through Jing would help
bootstrap liquidity on that venue.

As a resting maker, the vault pays Jing's 10 bps book fee and collects the
20 bps taker rebate when a taker fills it. In the liquidation phase the router
may route to third-party AMMs (Bitflow, Velar).

This is disclosed so the venue choice may be weighed accordingly. The DAO keeps
the ability to recall sBTC and to replace the vault by proposal.

## Rationale

**Why a maker-first order book instead of an RFQ desk?** An earlier draft of
this CCIP specified a request-for-quote desk, where the price is committed
before settlement. RFQ needs a market maker to respond to each request, and the
vault needs a way to recover if none does. The maker-first design removes the
counterparty dependency. The vault rests at the oracle mid, which any taker can
fill, and falls back to oracle-floored router sales only after the window
elapses. No one is needed to quote, no keeper key exists, and no caller can
choose the price.

**Why not an AMM swap alone?** A recurring swap of predictable size on a
predictable schedule against a public pool invites front-running. That is the
same failure the fair order book removes from the redemption side. Here, AMM
pools are used only after the maker phase, in capped chunks, behind an oracle
floor.

**Why not denominate the order book in sBTC?** Par is not a market price. It is
treasury arithmetic: a known quantity of STX behind a known supply of MIA,
verifiable by anyone. Nothing backs MIA in sBTC, so an sBTC par would have to be
derived from an external price rather than from the treasury. Holders would
redeem against an estimate instead of against the backing.

**Why not distribute sBTC directly to redeemers?** The same problem in another
place. It requires a MIA/sBTC price at distribution time, which must come from an
oracle.

**Why burn rather than seed liquidity?** Seeding a MIA/sBTC pool would create a
second venue for MIA and has real value. It also requires the DAO to hold and
manage a position, decide who supplies the paired side, and accept impermanent
loss. Burning requires no ongoing decision and benefits every holder in
proportion. If the community prefers the liquidity route, it should be proposed
separately.

**Why refresh par by proposal?** An automatic refresh would let par move while
offers rest, and sellers could no longer price against it. A proposal per
refresh keeps the timing a deliberate decision. The formula still fixes the
value, so approvers cannot choose a number.

## Decisions on Earlier Open Questions

The earlier draft left four questions open. The reference implementation decides
them as follows. Each can be changed later by proposal.

1. **Par refresh.** Set at activation, then refreshed by a separate Direct
   Execute proposal calling `update-par`, ideally between rounds.
2. **Spread.** Burned. The DAO is the only buyer and burns all MIA it acquires.
3. **Swap cadence.** One batch per funding. A 288-block maker window, then
   chunked router liquidation. Funding during an open window joins the batch.
4. **Venue.** The Jing market and router are named in the vault as constants.
   Changing venue requires a new vault and a proposal.

## Backwards Compatibility

`ccd013-burn-to-exit-mia` and the existing order book remain deployed and
readable. Historical redemptions are unaffected. Neither the ccd013 ratio nor
its treasury reader is changed.

Offers currently resting on the existing book are unaffected and may be
cancelled by their owners at any time. To use the new book, owners must cancel
and repost. Nothing migrates automatically.

## Activation

Execution through `ccd001-direct-execute` with 3 of 5 approver signals,
following the pattern established by CCIP-027. No tokenholder vote is held, and
the approver set, threshold and sunset are not changed.

The proposal `ccip028-activate-mia-fair-redemption` executes atomically:

1. Requires both `contract-caller` and `tx-sender` to be `base-dao`, and
   rejects a book whose par is already set.
2. Enables `ccd016-swap-vault-mia-v2` as a DAO extension. The book and the
   proposal are not enabled. The vault receives general DAO extension authority,
   not a token-scoped permission. Its code uses that authority only to call
   `withdraw-ft` for sBTC on the rewards treasury.
3. Calls `set-allowed` for `sbtc-token` on `ccd002-treasury-mia-rewards-v3`.
4. Sets the vault's launch parameters:

   | Setting                      | Value                       |
   | ---------------------------- | --------------------------- |
   | Maker window                 | 288 Bitcoin blocks          |
   | Placement leeway             | 1,000 bps (10%)             |
   | Router slippage floor        | 100 bps (1%)                |
   | Router chunk cap             | 1,000,000 sats (0.01 sBTC)  |
   | Maker spread                 | 1 bp over the mid           |
   | DIA agreement band           | 1,000 bps (10%)             |
   | Router cooldown              | 1 Bitcoin block             |
   | Emergency no-Pyth slippage   | 1,000 bps (10%), DAO only   |
   | Pyth fee budget              | 0 uSTX                      |

5. Sets the book's minimum offer to 100,000 MIA, calls `update-par`, and
   unpauses the book.

Activation moves no funds and performs no swap. Funding, placement,
liquidation, book funding and crossing are separate calls anyone can make, so
execution timing does not depend on approver availability. A runner or
community member still has to submit them. Price-bearing calls need a fresh
signed Pyth Lazer update.

If Pyth Lazer begins charging a fee, a proposal must raise the fee budget.
Otherwise the price-bearing calls will fail.

## Reference Implementation

Pull request: [citycoins/protocol#92](https://github.com/citycoins/protocol/pull/92)

- `contracts/extensions/ccd015-redemption-book-mia-stx.clar`
  (SHA-256 `b3526d4ce89e615108c1b6781bf67b6f8354e3880adb404883a30fbdd36ec65f`)
- `contracts/extensions/ccd016-swap-vault-mia-v2.clar`
  (SHA-256 `ac0c305f8bfcfd9f210fb37cba6e1f4d61a26ad40ed7ca4792beca948778502e`)
- `contracts/proposals/ccip028-activate-mia-fair-redemption.clar`
- `README-ccip028-activation.md`: deployment order, approver call and
  operational call sequence

Verification in the pull request:

- 8 Clarinet SDK integration cases for activation (`npm run test:ccip028`)
- Mainnet-fork (stxer) runs against the live DAO, Direct Execute, tokens and
  Jing contracts, including a full activation with simulated signals from three of the real approver
  principals followed by fund, place, reclaim, router swap, fuel, cross and
  burn (47/47 checks)
- Rendezvous invariant runs for the book and vault
- Several prior aibtc audit bounties on the book and vault, with findings and
  verdicts recorded in `contracts/extensions/README-ccd016-swap-vault-v2.md`.
  An audit bounty on the exact pull request is open until 2026-11-19.

Before approvers signal, the deployed bytes must match the hashes above, the
book's par must still be zero, and the open audit bounty should be concluded.
