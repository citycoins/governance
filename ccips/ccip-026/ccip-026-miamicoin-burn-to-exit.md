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
| Status        | Draft                                 |
| Created       | 2025-02-17                            |
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
   - Example estimate (as of cycle 126):
     - Mining Treasury Balance: ~10,241,497 STX
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

- The original city treasury (~10.2M STX) remains untouched and continues stacking
- The rewards treasury receives stacking payouts from two sources (per CCIP-012[^7]/CCIP-013[^8]):
  - Its own stacked STX (compounding)
  - The mining treasury's stacking payouts
- The rewards treasury currently contains ~1,217,764 STX (as of cycle 126)
- Upon activation, `revoke-delegate-stx` will be called to make the rewards treasury STX liquid for redemptions
- The rewards treasury will continue to be refilled by stacking payouts as long as the mining treasury remains stacked
- The extension will maintain a public record of all redemptions
- The burn-to-exit mechanism will remain enabled until the rewards treasury is empty
- The extension can be disabled through a separate governance proposal if needed
- Participation is optional, allowing holders to maintain positions if desired

## Backwards Compatibility

This CCIP supplements CCIP-020 and CCIP-024 by introducing a new, independent mechanism that does not alter the contracts or functionality established by the supplemented CCIPs.

## Activation

This CCIP will be voted on using a vote contract that adheres to CCIP-015 using the last two active stacking cycles for the protocol:

- MIA cycles 82 and 83

No scale factor is required.

The vote will:

- Only count MIA votes
- Be tallied and available in read-only functions
- Begin when the contract is deployed and continue for 2,016 Bitcoin blocks (approximately 2 weeks)

Upon successful vote (more yes votes than no votes):

1. The extension contract (ccd013-burn-to-exit-mia) will be enabled in the DAO
2. The `initialize-redemption` function will be called to start redemptions
3. The redemption ratio will be locked based on the mining treasury balance and total MIA supply

## Reference Implementations

- CCIP-026 Proposal: ccip026-miamicoin-burn-to-exit.clar[^5]
- CCD013 Burn Extension: ccd013-burn-to-exit-mia.clar[^6]

## Footnotes

[^1]: https://github.com/citycoins/governance/blob/main/ccips/ccip-024/ccip-024-miamicoin-community-signal-vote.md
[^2]: https://github.com/citycoins/governance/blob/main/ccips/ccip-020/ccip-020-graceful-protocol-shutdown.md
[^3]: https://explorer.hiro.so/txid/SP8A9HZ3PKST0S42VM9523Z9NV42SZ026V4K39WH.ccd002-treasury-mia-rewards-v3?chain=mainnet
[^4]: https://explorer.hiro.so/txid/SP8A9HZ3PKST0S42VM9523Z9NV42SZ026V4K39WH.ccd002-treasury-mia-mining-v3?chain=mainnet
[^5]: https://github.com/citycoins/protocol/blob/fix/implement-ccip-026/contracts/proposals/ccip026-miamicoin-burn-to-exit.clar
[^6]: https://github.com/citycoins/protocol/blob/fix/implement-ccip-026/contracts/extensions/ccd013-burn-to-exit-mia.clar
[^7]: https://github.com/citycoins/governance/blob/main/ccips/ccip-012/ccip-012-stabilize-emissions-and-treasuries.md
[^8]: https://github.com/citycoins/governance/blob/main/ccips/ccip-013/ccip-013-stabilize-protocol-and-simplify-contracts.md
