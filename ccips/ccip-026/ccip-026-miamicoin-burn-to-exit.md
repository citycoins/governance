# CCIP-026

## Preamble

| CCIP Number   | 026                                   |
| ------------- | ------------------------------------- |
| Title         | MiamiCoin Burn to Exit                |
| Author(s)     | Jason Schrader jason@joinfreehold.com |
| Consideration | Governance, Economic                  |
| Type          | Standard                              |
| Status        | Draft                                 |
| Created       | 2025-02-17                            |
| License       | BSD-2-Clause                          |
| Supplements   | CCIP-020, CCIP-024                    |

## Introduction

Following the MiamiCoin community signal vote[^1] and the graceful protocol shutdown[^2], this CCIP proposes implementing a burn-to-exit mechanism for MIA holders using the secondary treasury[^3]. This mechanism will provide MIA holders with an option to burn their tokens in exchange for STX at a fixed ratio, while leaving the original city treasury[^4] untouched.

## Specification

### Burn-to-Exit Mechanism

The implementation will create a new extension that:

1. can access the approximately 400,000 STX secondary treasury while enabled

   - value hard-coded preventing access to the original 10.2M treasury
   - burn to exit can be disabled through a separate proposal

2. implement a fixed redemption ratio

   - 1,700 STX for every 1,000,000 MiamiCoin (MIA)
   - Ratio calculation:
     - Original City Treasury Balance: ~10,241,497 STX
     - Total MIA Supply: ~5,988,905,152 MIA
     - Ratio = (10,241,497 * 1,000,000) / 5,988,905,152 = ~1,700 STX per 1M MIA

3. Functionality:

   - Any MIA holder can participate
   - No minimum or maximum redemption amount
   - MIA tokens will be burned upon redemption
   - STX will be transferred to the participant's wallet
   - Contract will continue functioning as long as enabled

### Technical Implementation

The extension will:

- Accept MIA tokens from users
- Calculate the STX return based on the fixed ratio
- Burn the received MIA tokens
- Transfer the calculated STX to the user
- Track total MIA burned and STX distributed
- Include read-only functions to check available STX balance
- Include read-only functions to check addresses, redemptions
- Return ERR_NOT_ENOUGH_FUNDS_IN_CONTRACT if redemption would exceed available STX

> [!NOTE]
> The burn-to-exit mechanism is one-way and irreversible. Once MIA tokens are burned, they cannot be recovered. Users should verify the contract's available STX balance through the read-only functions before burning tokens.

### Treasury Management

- The original city treasury (~10.2M STX) remains untouched
- Stacking rewards from the original treasury will continue same as before
- The extension will maintain a public record of all redemptions

## Backwards Compatibility

This CCIP supplements CCIP-020 and CCIP-024 but is not backwards compatible.

## Activation

This CCIP will be voted on using a vote contract that adheres to CCIP-015 using the last two active stacking cycles for the protocol:

- MIA cycles 82 and 83
- NYC cycles 82 and 83

No scale factor is required.

The vote will:
- Only count MIA votes
- Be tallied and available in read-only functions
- Require 25% of MIA participation to be considered valid
- Begin when the contract is deployed and continue for 2016 Bitcoin blocks

Upon successful vote:
1. The extension contract will be deployed
2. The extension will be granted access to the secondary treasury
3. The burn-to-exit mechanism will be enabled

## Reference Implementations

- CCIP-026 Proposal: ccip026-miamicoin-burn-to-exit.clar[^5]
- CCD013 Burn Extension: ccd013-burn-to-exit-mia.clar[^6]

## Footnotes

[^1]: https://github.com/citycoins/governance/blob/main/ccips/ccip-024/ccip-024-miamicoin-community-signal-vote.md
[^2]: https://github.com/citycoins/governance/blob/main/ccips/ccip-020/ccip-020-graceful-protocol-shutdown.md
[^3]: https://explorer.hiro.so/txid/SP8A9HZ3PKST0S42VM9523Z9NV42SZ026V4K39WH.ccd002-treasury-mia-mining-v2?chain=mainnet
[^4]: https://explorer.hiro.so/txid/SP8A9HZ3PKST0S42VM9523Z9NV42SZ026V4K39WH.ccd002-treasury-mia-city?chain=mainnet
[^5]: https://github.com/citycoins/protocol/blob/fix/implement-ccip-026/contracts/proposals/ccip026-miamicoin-burn-to-exit.clar
[^6]: https://github.com/citycoins/protocol/blob/fix/implement-ccip-026/contracts/extensions/ccd013-burn-to-exit-mia.clar
