# CCIP-023

## Preamble

| CCIP Number   | 023                                                  |
| ------------- | ---------------------------------------------------- |
| Title         | CityCoins Treasury Redemption MIA                    |
| Author(s)     | Raphael R. Sierra rapha@fontainebleau-management.com |
|               | Jason Schrader jason@joinfreehold.com                |
|               | Friedger Müffke mail@friedger.de                     |
| Consideration | Governance                                           |
| Type          | Standard                                             |
| Status        | Draft                                                |
| Created       | 2024-07-16                                           |
| License       | BSD-2-Clause                                         |
| Supplements   | CCIP-020                                             |

## Introduction

With the execution of CCIP-020 Graceful Protocol Shutdown[^1] the functions for mining and stacking CityCoins are disabled.

Following that, this CCIP proposes a redemption mechanism for MIA token holders to be fairly compensated by redeeming their MIA for a portion of the STX held in the MIA mining treasury[^2].

It is assumed that this is also in the interest of the city of Miami (under silence procedure). If there are concerns the redemption can be stopped within 8000 bitcoin blocks (approx. 2 months) after the vote about this CCIP has been closed.

## Specification

### End of Delegated Stacking

If delegated stacking was enabled, then stacking is revoked.

Once stacked STX are unlocked, the redemption is started.

### Redemption Extension

This CCIP will implement a new redemption extension for MIA.

The redemption extension will:

- receive and hold funds from the MIA mining treasury contract[^2]
- record the combined total supply of MIA at the time of initialization
- calculate the ratio of MIA to STX based on the total supply and contract balance
- allow users to burn MIA in exchange for their portion of STX
- track and return the redemption information for the contract and its users

### Redemption Mechanism

The MIA mining treasury[^2] currently holds 10.9M STX and upon successful execution of this proposal and unlock of stacked STX the entire balance will be transferred to the redemption extension.

Following the transfer and upon initialization from the proposal, the redemption extension will:

- will allow a new proposal without vote using direct exection to revoke the redemption based on requests from Miami government within 8000 bitcoin blocks.
- will be activated automatically at 8000 bitcoin blocks after the execution of the proposal
- at activation of the remption, the extension will
  - get the total supply for MIA V1[^3] and MIA V2[^4]
  - add the total supplies accounting for decimal differences
    - MIA V1 has 0 decimals
    - MIA V2 has 6 decimals
  - get the balance of the redemption contract itself
  - set variables representing all data gathered above
  - create a redemption ratio from `balance / total supply`
  - set a variable that redemption is enabled, allowing access

The redemption extension then exposes a public function that:

- calculates amount to transfer based on `balance * redemption ratio`
- verify that redemption is enabled
- verify there is something to burn (MIA V1 or V2)
- verify there is something to redeem
- burn the MIA balance for the user
- transfer the redemption STX to the user from the contract
- store redemption info for the user and the contract
- print the redemption info from the read only functions

## Backwards Compatibility

This CCIP is supplemental to CCIP-020[^1] and not backwards compatible.

## Activation

This CCIP will be voted on using a vote contract that adheres to CCIP-015[^5] using the last two active stacking cycles for the protocol.

- MIA cycles 82 and 83
- NYC cycles 82 and 83

No scale factor is required.

Voting will begin when the contract is deployed and continue until the proposal is executed.

This CCIP follows the same voting format as CCIP-020[^1] except only MIA votes are counted.

- the votes are tallied and available in read-only functions
- the proposal will not pass unless 25% of MIA have participated

## Reference Implementations

- CCIP-023 Proposal: ccip022-citycoins-treasury-redemption-mia.clar
- CCD-012 Redemption Extension: ccd012-redemption-mia.clar[^7]

## Footnotes

[^1]: https://github.com/citycoins/governance/blob/main/ccips/ccip-020/ccip-020-graceful-protocol-shutdown.md
[^2]: https://explorer.hiro.so/txid/SP8A9HZ3PKST0S42VM9523Z9NV42SZ026V4K39WH.ccd002-treasury-mia-mining-v2?chain=mainnet
[^3]: https://explorer.hiro.so/txid/SP466FNC0P7JWTNM2R9T199QRZN1MYEDTAR0KP27.miamicoin-token?chain=mainnet
[^4]: https://explorer.stacks.co/txid/SP1H1733V5MZ3SZ9XRW9FKYGEZT0JDGEB8Y634C7R.miamicoin-token-v2?chain=mainnet
[^5]: https://github.com/citycoins/governance/blob/main/ccips/ccip-015/ccip-015-community-proposal-voting-process.md
