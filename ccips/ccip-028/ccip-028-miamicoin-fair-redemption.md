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

Measured on-chain on 2026-11-06 (Stacks block 9,427,614, Bitcoin block 974,810):

| Input                                     | Value                         |
| ----------------------------------------- | ----------------------------- |
| MIA v1 supply                             | 260,130,901 MIA               |
| MIA v2 supply                             | 4,754,074,853.491373 MIA      |
| Combined supply                           | 5,014,205,754.491373 MIA      |
| `ccd002-treasury-mia-mining-v3`           | 0 STX                         |
| `ccd014-pox5-staking-mia` (locked)        | 10,241,497.066794 STX         |
| Treasury, total                           | 10,241,497.066794 STX         |
| Ratio by the DAO's formula                | 2,042.50 STX per 1M MIA       |
| Ratio currently paid by ccd013            | 1,710 STX per 1M MIA          |

The formula gives 19.4% more than holders are currently paid. The inputs are
unchanged from the first measurement on 2026-08-13: no MIA has been burned
since, which is consistent with `ccd013` reading an empty mining treasury.
The figure will move with every burn once this proposal is active, so what
should be ratified is the formula, not the number above.

The ratio cannot be corrected in place. `ccd013.initialize-redemption` sets
`redemptions-enabled` permanently and the contract exposes no setter, so 1710 is
final there. Its treasury reader also still points at
`ccd002-treasury-mia-mining-v3`, which CCIP-027 emptied into
`ccd014-pox5-staking-mia`. It reads 0 STX and would revert with
`ERR_GETTING_REDEMPTION_BALANCE`. A corrected par therefore needs a new
contract, not a configuration change.

## Specification

### Flow of funds

```
  pox-5 rewards (sBTC)
          |
          v
  ccd014-pox5-staking-mia
          |  forward-rewards                 (existing, permissionless)
          v
  ccd002-treasury-mia-rewards-v3
          |  sBTC, whole balance              (new permission, see Activation)
          v
  ccd016-swap-vault-mia-v2  <---->  Jing market / smart router
          |  STX, whole balance               (sBTC out, STX back to the vault)
          v
  ccd015-redemption-book-mia-stx
          |  STX to sellers, at or below par
          v
      MIA v2 burned
```

Every arrow is a fixed destination. No caller chooses where funds go, and no
caller chooses an amount: each step moves the whole available balance, or a
bounded chunk of it.

The staked treasury in `ccd014-pox5-staking-mia` is not touched. Only staking
rewards flow through this path.

### Contracts and permissions

| Contract                          | New | DAO extension | Holds            |
| --------------------------------- | --- | ------------- | ---------------- |
| `ccd002-treasury-mia-rewards-v3`  | no  | yes (exists)  | sBTC rewards     |
| `ccd016-swap-vault-mia-v2`        | yes | yes (enabled by this CCIP) | sBTC being sold, STX proceeds |
| `ccd015-redemption-book-mia-stx`  | yes | no            | STX budget, MIA v2 offered by sellers |

The swap vault is the only contract that gains DAO extension rights. It needs
them to withdraw sBTC from the rewards treasury. Extension rights are not
limited to one token or one treasury, so the vault's code is part of what is
being ratified. That code uses the rights only to withdraw sBTC from
`ccd002-treasury-mia-rewards-v3`.

The book is not an extension. It can spend only what it holds.

Functions marked "DAO" below can be called only by `base-dao` or an enabled
extension, which in practice means an executed proposal.

### The Book: `ccd015-redemption-book-mia-stx`

The book is based on the deployed fair book
`SPV9K21TBFAK4KNRJXF5DFP8N7W46G4V9RCJDC22.mia-fair-faktory-v2`, in production
against `ccd013-burn-to-exit-mia` since cycle 138 (see Related Work). It keeps
that book's offer mechanics: a sorted list of sell offers, cheapest first,
partial fills, and cancellation at any time.

It differs from the deployed book in three ways:

- **The DAO is the only buyer.** The deployed book lets a settler bring STX and
  receive MIA. This book spends only STX it already holds, which comes from the
  swap vault.
- **All MIA bought is burned.** Nothing is delivered to a settler and no spread
  is retained.
- **Asks and par are both in STX.** No price oracle is involved in the book.

**Funds in.** STX from the swap vault. MIA v2 from sellers posting offers.

**Funds out.** STX only to sellers whose offers are filled, at their asks. MIA
only back to its owner (on cancel or eviction) or into a burn. There is no
function that withdraws STX, including for the DAO. STX that cannot be matched
against eligible offers stays in the book until such offers arrive.

**Main functions.**

| Function        | Who    | Effect |
| --------------- | ------ | ------ |
| `place-offer`   | anyone | Deposits MIA v2 with an ask in STX. The ask must be at or below par. |
| `change-offer`  | owner  | Changes the ask, or adds MIA to the offer. Still at or below par. |
| `cancel-offer`  | owner  | Returns the offered MIA. Works even while the book is paused. |
| `cross-book`    | anyone | Spends the book's STX on offers at or below par, cheapest first, and burns the MIA bought. |
| `update-par`    | DAO    | Writes par from the formula below. Takes no value as input. |
| `set-paused`    | DAO    | Pauses or resumes offers and crossing. |
| `set-min-deposit` | DAO  | Sets the minimum offer size. |

**Limits.** At most 50 offers and one offer per address. Each offer is between
the minimum (100,000 MIA at activation) and 10,000,000 MIA. When the book is
full, a new offer must be cheaper per MIA than the worst resting offer, and the
worst offer is refunded to its owner. Only MIA v2 is accepted. Par counts v1
supply as well, so v1 holders must convert to v2 before offering.

### Par

Par is the DAO's backing per MIA, stored in the book as a snapshot:

```
par = STX held by ccd002-treasury-mia-mining-v3 and ccd014-pox5-staking-mia
      (locked and unlocked)
      / (MIA v1 supply + MIA v2 supply)
```

Only the DAO can update par, and the update takes no value: it writes whatever
the formula returns at that moment. A proposal decides **when** par moves,
never **what** it moves to.

Par is set once at activation. Each later refresh is a separate Direct Execute
proposal, ideally between rounds when the book has been crossed, so that par
stays fixed while sellers price against it. Offers are checked against par when
placed and again when crossed, so a refresh never fills an offer above the new
par.

Each fill burns MIA worth `par * amount` of the treasury while paying only the
seller's ask. Payment comes from staking rewards, not from the treasury, so the
treasury `T` is unchanged and only supply `S` shrinks:

```
par_before = T / S           par_after = T / (S - burned)
```

The retired claim accrues to the holders who did not sell. Cheaper asks retire
more MIA per STX of rewards, which is why the book fills cheapest first.

There is no fixed-ratio redemption in this design. A holder exits by selling
into the book at or below par.

### The Swap Vault: `ccd016-swap-vault-mia-v2`

The vault converts the sBTC rewards to STX and sends the STX to the book.

**Funds in.** sBTC, only from `ccd002-treasury-mia-rewards-v3`.

**Funds out.**

- STX only to the book.
- sBTC only to the Jing sBTC/STX market and smart router while selling, or back
  to `ccd002-treasury-mia-rewards-v3` by proposal. sBTC resting on the Jing
  market remains the vault's position and can only be reclaimed to the vault.
- A configurable amount of STX is kept back to pay oracle fees, if any (0 at
  activation).

**Pricing.** Callers never set a price. Every sale is bounded by the BTC/STX
mid from a signed Pyth Lazer update, which is verified on-chain and must agree
with the DIA on-chain rate within a band (10% at activation). A caller can only
choose when to act.

**Selling.** Each funding of an empty vault opens a batch with a window of
288 Bitcoin blocks (about two days). sBTC arriving during the window joins the
batch.

1. While the window is open, the vault's sBTC can only rest on the Jing order
   book as an ask 1 bp above the mid. It is never sold below the mid in this
   phase, and the ask switches off if the mid falls more than 10% below its
   level at placement.
2. After the window, any unsold sBTC can be reclaimed and sold through the Jing
   smart router (Jing book, Bitflow, Velar) at no less than 1% below the mid.
   Each sale is at most 0.01 sBTC, with at least one Bitcoin block between
   sales. Anything no venue takes above that floor stays in the vault.

**Main functions.**

| Function             | Who    | Effect |
| -------------------- | ------ | ------ |
| `fund-from-treasury` | anyone | Moves the rewards treasury's whole sBTC balance into the vault. |
| `jing-place`         | anyone | During the window: rests the vault's sBTC on the Jing book at mid + 1 bp. |
| `jing-reclaim`       | anyone | After the window: returns sBTC from the Jing market to the vault. |
| `router-swap`        | anyone | After the window: sells one chunk through the smart router at the floor. |
| `fuel-fair-book`     | anyone | Sends the vault's STX, minus the oracle-fee reserve, to the book. |
| `close-batch`        | anyone | Clears the batch clock once the vault is empty. |
| `dao-recall-sbtc`    | DAO    | Returns all sBTC held by the vault to the rewards treasury. |
| `dao-reclaim`        | DAO    | Returns sBTC from the Jing market to the vault at any time. |
| `jing-refloor`, `jing-take`, `router-swap-split`, `router-swap-split-dia` | DAO | Manual selling tools, including an emergency mode priced from DIA only. All prices stay oracle-bound. |
| `set-*`              | DAO    | Changes window, leeway, slippage, chunk size, spread, DIA band, cooldown and oracle-fee reserve, each within a hard-coded maximum. |

**Front-running exposure.** The resting ask fills at the oracle mid and cannot
be sandwiched. Router sales do touch AMM pools. The floor caps what a sandwich
can extract at 1% of a 0.01 sBTC chunk, and the attacker pays the venue fee on
both legs, which makes it unprofitable on Bitflow and limits it to about 0.4%
of a chunk on Velar. These bounds depend on chunk size, slippage and venue fees,
and should be revisited if those change.

## Disclosure

The swap vault trades on the Jing sBTC/STX market and smart router
(`SPV9K21TBFAK4KNRJXF5DFP8N7W46G4V9RCJDC22.markets-sbtc-stx-jing-v6-4` and
`.swap-router-sbtc-stx-jing-v5-5`), deployed from the same principal as the
production fair book. CityCoins volume routed through Jing would help
bootstrap liquidity on that venue.

As a resting maker, the vault pays Jing's 10 bps book fee and collects the
20 bps taker rebate when a taker fills it. In the liquidation phase the router
may route to third-party AMMs (Bitflow, Velar).

This is disclosed so the venue choice may be weighed accordingly. The DAO keeps
the ability to recall sBTC and to replace the vault by proposal.

## Rationale

The earlier draft of this CCIP left four questions open. They are decided as
follows, and each can be changed later by proposal.

**Par refresh: formula-locked, timed by proposal.** Par is set at activation
and refreshed by separate Direct Execute proposals, ideally between rounds. An
automatic refresh would move par while offers rest, and sellers could no longer
price against it. Because the value always comes from the formula, approvers
decide the timing but cannot choose a number.

**Spread: burned.** The DAO is the only buyer and burns all MIA it acquires.
Seeding a MIA/sBTC pool instead would create a second venue for MIA and has real
value. It also requires the DAO to hold and manage a position, decide who
supplies the paired side, and accept impermanent loss. Burning requires no
ongoing decision and benefits every holder in proportion. If the community
prefers the liquidity route, it should be proposed separately.

**Swap cadence: one batch per funding.** Each funding opens a two-day window in
which the sBTC rests on an order book at the oracle mid, followed by capped
router sales. Funding during an open window joins the batch.

**Venue: a maker-first order book with a router fallback, not an RFQ desk.** The
earlier draft specified a request-for-quote desk, where the price is committed
before settlement. RFQ needs a market maker to respond to each request, and a
recovery path if none does. Resting at the oracle mid removes that dependency:
any taker can fill it, no keeper key exists, and no caller can choose the
price. A plain AMM swap of predictable size on a predictable schedule invites
front-running, the failure the fair order book removes from the redemption
side, so AMM pools are used only after the window, in capped chunks, behind an
oracle floor. The venue is fixed in the vault; changing it needs a new vault and
a proposal.

**Why the book is denominated in STX.** Par is not a market price. It is
treasury arithmetic: a known quantity of STX behind a known supply of MIA,
verifiable by anyone. Nothing backs MIA in sBTC, so an sBTC par would have to be
derived from an external price rather than from the treasury.

**Why rewards are not distributed as sBTC directly.** It would require a
MIA/sBTC price at distribution time, which must come from an oracle, and moves
the pricing problem rather than removing it.

## Backwards Compatibility

`ccd013-burn-to-exit-mia` and the existing order book remain deployed and
readable. Historical redemptions are unaffected. Neither the ccd013 ratio nor
its treasury reader is changed.

Offers currently resting on the existing book are unaffected and may be
cancelled by their owners at any time. To use the new book, owners must cancel
and repost. Nothing migrates automatically.

## Activation

Execution through `ccd001-direct-execute` with 3 of 5 approver signals because this proposal is the continuation of CCIP-026 after CCIP-027.

No tokenholder vote is held. The approver set, threshold and sunset of
`ccd001-direct-execute` are not changed.

Activation enables the swap vault as an extension and allows to move sBTC from the MIA rewards treasury. It also sets launch parameters for the vault and the book.

Activation does not move any funds and performs no swap.

### What the proposal does when executed

When the third approver signals, `ccd001-direct-execute` executes
`ccip028-activate-mia-fair-redemption` through `base-dao`. In one transaction,
the proposal:

1. Checks that it is being executed by `base-dao` and that the book's par has
   not been set yet. Otherwise it fails and nothing changes.
2. Enables `ccd016-swap-vault-mia-v2` as a DAO extension. This is the only new
   extension.
3. Allows sBTC on `ccd002-treasury-mia-rewards-v3`. CCIP-027 already did this,
   so this step only confirms it.
4. Sets the swap vault's launch parameters:

   | Setting                      | Value                       |
   | ---------------------------- | --------------------------- |
   | Batch window                 | 288 Bitcoin blocks          |
   | Leeway for the resting ask   | 10% below the mid           |
   | Router floor                 | 1% below the mid            |
   | Router chunk                 | 0.01 sBTC                   |
   | Resting ask spread           | 1 bp above the mid          |
   | Pyth / DIA agreement band    | 10%                         |
   | Router cooldown              | 1 Bitcoin block             |
   | Emergency DIA-only floor     | 10% below the DIA rate      |
   | Oracle-fee reserve           | 0 STX                       |

5. Sets the book's minimum offer to 100,000 MIA.
6. Sets par from the formula (2,042.50 STX per 1,000,000 MIA as measured on
   2026-11-06; the value at execution applies).
7. Unpauses the book.

If any step fails, the whole execution is reverted and the proposal can be
executed again. After activation, every step in the flow of funds is a call
anyone can make. Someone still has to submit those calls; the price-bearing
vault calls need a fresh signed Pyth Lazer update.

## Related Work

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

That implementation lets a settler buy and retains the spread. The new book
replaces the settler with the DAO and burns everything.

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
