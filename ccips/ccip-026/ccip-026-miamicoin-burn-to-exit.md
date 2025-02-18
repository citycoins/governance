# CCIP-026

## Preamble

| CCIP Number   | 026                                   |
| ------------- | ------------------------------------- |
| Title         | MiamiCoin Burn to Exit               |
| Author(s)     | Jason Schrader jason@joinfreehold.com |
| Consideration | Governance, Economic                  |
| Type          | Standard                              |
| Status        | Draft                                |
| Created       | 2025-02-17                           |
| License       | BSD-2-Clause                          |
| Supplements   | CCIP-020, CCIP-024                    |

## Introduction

Following the MiamiCoin community signal vote (CCIP-024) and the graceful protocol shutdown (CCIP-020), this CCIP proposes implementing a burn-to-exit mechanism for MIA holders using the secondary treasury containing approximately 400,000 STX. This mechanism will provide MIA holders with an option to burn their tokens in exchange for STX at a fixed ratio, while leaving the primary city treasury untouched.

## Specification

### Burn-to-Exit Mechanism

The implementation will create an isolated extension with the following characteristics:

1. Initial Funding:
   - Transfer of approximately 400,000 STX from the secondary treasury
   - Additional STX from ongoing stacking rewards from the original 10.2M treasury

2. Redemption Ratio:
   - 1,700 STX for every 1,000,000 MiamiCoin (MIA)
   - This ratio is derived from the current relationship between total MIA supply and the original treasury of 10.2M STX

3. Functionality:
   - Any MIA holder can participate
   - No minimum or maximum redemption amount
   - MIA tokens will be burned upon redemption
   - STX will be transferred to the participant's wallet
   - Contract will continue functioning as long as STX remains available

### Technical Implementation

The extension will:
- Accept MIA tokens from users
- Calculate the STX return based on the fixed ratio
- Burn the received MIA tokens
- Transfer the calculated STX to the user
- Track total MIA burned and STX distributed
- Include read-only functions to check available STX balance
- Continue receiving STX from stacking rewards

### Treasury Management

- The original city treasury (~10.2M STX) remains untouched
- Stacking rewards from the original treasury will continue flowing to this extension
- The extension will maintain a public record of all redemptions

## Backwards Compatibility

This CCIP supplements CCIP-020 and CCIP-024 but is not backwards compatible.

## Activation

This CCIP will be implemented as a new extension contract that:

1. Receives the initial 400,000 STX funding
2. Implements the burn-to-exit mechanism
3. Continues receiving stacking rewards
4. Requires no vote as it uses only the secondary treasury

## Reference Implementations

- ccip026-miamicoin-burn-to-exit contract (to be implemented)
- Extension deployment and STX transfer contracts (to be implemented)

## Footnotes

[^1]: CCIP-020 Graceful Protocol Shutdown
[^2]: CCIP-024 MiamiCoin Community Signal Vote
