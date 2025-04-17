# CCIP-016

## Preamble

| CCIP Number   | 016                                                  |
| ------------- | ---------------------------------------------------- |
| Title         | Refund Incorrect CCD007 Payouts                      |
| Author(s)     | Jason Schrader jason@joinfreehold.com                |
|               | Raphael R. Sierra rapha@fontainebleau-management.com |
| Consideration | Governance, Economic                                 |
| Type          | Standard                                             |
| Status        | Draft                                                |
| Created       | 2023-04-26                                           |
| License       | BSD-2-Clause                                         |

## Introduction

The current stacking contract `ccd007-citycoin-stacking`[^1] has a bug that incorrectly returns 0 STX rewards for a cycle if the following conditions are true:

- the cycle has just ended or the cycle has id #83
- the cycle is not paid by the pool operator
- the user has CityCoins to be returned

When the above conditions are met, the contract does not emit an error and instead calculates the STX reward as 0.

This CCIP will analyze any incorrectly assigned rewards since stacking started in this contract at cycle 54 until cycle 83 when stacking was disabled, then transfer the correct STX reward to each affected user.

## Specification

### Research

In order to analyze what users were affected, we need to determine the window in Bitcoin/Stacks block heights that the bug was active in each cycle, then analyze the related transactions to determine who claimed during this period.

For the cycle payouts, we need to look at transactions sent to `ccd011-stacking-payouts` calling the functions `send-stacking-reward-mia` and `send-stacking-reward-nyc`.

For the claims, we need to look at transactions emitting the print event `stacking-claim` from contract `ccd007-citycoin-stacking` that meet the above mentioned criteria.

#### Cycle Block Heights

| Cycle | BTC Start Height | BTC End Height | STX Start Height | STX End Height |
| ----- | ---------------- | -------------- | ---------------- | -------------- |
| 54    | 779,450          | 781,550        | 97,453           | 99,112         |
| 55    | 781,550          | 783,650        | 99,112           | 100,670        |
| 56    | 783,650          | 785,750        | 100,670          | 102,356        |
| 57    | 785,750          | 787,850        | 102,356          | 104,060        |
| 58    | 787,850          | 789,950        | 104,060          | 105,687        |
| 59    | 789,950          | 792,050        | 105,687          | 107,473        |
| 60    | 792,050          | 794,150        | 107,473          | 109,189        |
| 61    | 794,150          | 796,250        | 109,189          | 110,979        |
| 62    | 796,250          | 798,350        | 110,979          | 112,809        |
| 63    | 798,350          | 800,450        | 112,809          | 114,689        |
| 64    | 800,450          | 802,550        | 114,689          | 116,486        |
| 65    | 802,550          | 804,650        | 116,486          | 118,283        |
| 66    | 804,650          | 806,750        | 118,283          | 120,101        |
| 67    | 806,750          | 808,850        | 120,101          | 121,908        |
| 68    | 808,850          | 810,950        | 121,908          | 123,725        |
| 69    | 810,950          | 813,050        | 123,725          | 125,515        |
| 70    | 813,050          | 815,150        | 125,515          | 127,251        |
| 71    | 815,150          | 817,250        | 127,251          | 128,894        |
| 72    | 817,250          | 819,350        | 128,894          | 130,575        |
| 73    | 819,350          | 821,450        | 130,575          | 132,196        |
| 74    | 821,450          | 823,550        | 132,196          | 133,807        |
| 75    | 823,550          | 825,650        | 133,807          | 135,513        |
| 76    | 825,650          | 827,750        | 135,513          | 137,300        |
| 77    | 827,750          | 829,850        | 137,300          | 138,938        |
| 78    | 829,850          | 831,950        | 138,938          | 140,625        |
| 79    | 831,950          | 834,050        | 140,625          | 142,301        |
| 80    | 834,050          | 836,150        | 142,301          | 143,989        |
| 81    | 836,150          | 838,250        | 143,989          | 145,643        |
| 82    | 838,250          | 840,350        | 145,643          | 147,282        |
| 83    | 840,350          | 842,450        | 147,282          | 149,154        |

#### Cycle Payout Data

| Cycle | MIA Payout Height | MIA Height Diff | MIA Payout Amount | NYC Payout Height | NYC Height Diff | NYC Payout Amount |
| ----- | ----------------- | --------------- | ----------------- | ----------------- | --------------- | ----------------- |
| 54    | 99,308            | 196             | 21,738.856765 STX | 99,308            | 196             | 32,957.636028 STX |
| 55    | 100,875           | 205             | 24,130.341569 STX | 100,875           | 205             | 36,583.295214 STX |
| 56    | null              | null            | null              | null              | null            | null              |
| 57    | null              | null            | null              | null              | null            | null              |
| 58    | null              | null            | null              | null              | null            | null              |
| 59    | null              | null            | null              | null              | null            | null              |
| 60    | 109,338           | 149             | 30,143.370950 STX | 109,338           | 149             | 45,695.032162 STX |
| 61    | 111,097           | 118             | 24,002.944424 STX | 111,097           | 118             | 36,386.519532 STX |
| 62    | 112,820           | 11              | 22,341.600977 STX | 112,820           | 11              | 33,867.164794 STX |
| 63    | 114,755           | 66              | 19,641.588871 STX | 114,755           | 66              | 29,773.591836 STX |
| 64    | 116,596           | 110             | 16,622.163699 STX | 116,596           | 110             | 25,195.947330 STX |
| 65    | 118,392           | 109             | 18,031.083017 STX | 118,392           | 109             | 27,330.659232 STX |
| 66    | 120,440           | 339             | 21,136.466347 STX | 120,440           | 339             | 32,036.545780 STX |
| 67    | 122,286           | 378             | 14,547.584045 STX | 122,286           | 378             | 22,049.542417 STX |
| 68    | 124,081           | 356             | 15,092.603850 STX | 124,081           | 356             | 22,875.310501 STX |
| 69    | 126,031           | 516             | 14,744.214112 STX | 126,031           | 516             | 22,346.829090 STX |
| 70    | 127,254           | 3               | 14,562.588849 STX | 127,254           | 3               | 22,070.459358 STX |
| 71    | 129,309           | 415             | 15,933.649223 STX | 129,309           | 415             | 24,147.184987 STX |
| 72    | 130,893           | 318             | 13,140.642834 STX | 130,893           | 318             | 19,913.782531 STX |
| 73    | 132,588           | 392             | 10,225.347415 STX | 132,680           | 484             | 15,495.109117 STX |
| 74    | 134,191           | 384             | 13,547.686726 STX | 134,191           | 384             | 20,528.917568 STX |
| 75    | 135,730           | 217             | 28,962.280296 STX | 135,730           | 217             | 43,886.090272 STX |
| 76    | 137,331           | 31              | 15,681.210555 STX | 137,331           | 31              | 23,761.013666 STX |
| 77    | 138,972           | 34              | 14,428.419333 STX | 138,972           | 34              | 21,862.239159 STX |
| 78    | 140,709           | 84              | 8,354.173172 STX  | 140,709           | 84              | 12,658.202794 STX |
| 79    | 142,306           | 5               | 9,695.916112 STX  | 142,306           | 5               | 14,691.143091 STX |
| 80    | 144,186           | 197             | 7,891.090017 STX  | 144,186           | 197             | 11,956.403387 STX |
| 81    | 145,685           | 42              | 5,606.386687 STX  | 145,685           | 42              | 8,494.571191 STX  |
| 82    | 147,735           | 453             | 5,912.068120 STX  | 147,735           | 453             | 8,958.011022 STX  |
| 83    | 149,420           | 266             | 30,599.097204 STX | 149,420           | 266             | 46,365.579712 STX |

### Analysis

Most claims are done by calling the function `claim-stacking-reward` of `ccd007-citycoin-stacking`.

There are two contracts that call this function during deployment: `SPN4Y5QPGQA8882ZXW90ADC2DHYXMSTN8VAR8C3X.claim-many` and `SPN4Y5QPGQA8882ZXW90ADC2DHYXMSTN8VAR8C3X.claim-many-nyc`. However, non of the calls to `claim-stacking-reward` meet the criteria. Therefore, these contracts can be ignored.

All direct function calls to `claim-stacking-reward` are filtered per cycle to meet the criteria mentioned above. The filter is encoded in typescript as follows:

```
tx.tx_status === "success" && // only successful transactions
tx.tx_type === "contract_call" && //
tx.contract_call.function_name === "claim-stacking-reward" && // only claims
tx.block_height &&
(cycleNumber === 83 || tx.block_height > cycleEndStxHeight) && // only claims that happened after the end of the cycle or cycle 83
((tx.contract_call.function_args![0].repr === '"mia"' &&
  tx.block_height < miaPayoutHeight) ||
(tx.contract_call.function_args![0].repr === '"nyc"' &&
  tx.block_height < nycPayoutHeight)) && // only claims that happened before the payout height
tx.contract_call.function_args && tx.contract_call.function_args[1].repr === `u${cycleNumber}` // only claims for the cycle in question
```

For each relevant claim, the user's amount of stacked CC is retrieved at the block height of the claim - 1. This is done by calling the `get-stacker` function on the stacking contract with the user id of the user. Note, that the stacked CC amount is set to 0 after the claim function.

The missed payout amount for each claim is calculated by multiplying the total payout amount of the cycle with the user's amount of stacked CC and then dividing by the total amount of stacked CC for the cycle.

The analysis is done with the scripts defined here: https://github.com/citycoins/scripts/pull/19

The scripts create clarity contract code that is used in the proposal for thie CCIP.

## Backwards Compatibility

None, this is a one-time correction.

## Activation

This CCIP will be voted on using a vote contract that adheres to CCIP-015[^3] using the last two active cycles from when the contract is deployed.

Currently, this would be:

- MIA cycles 82 and 83
- NYC cycles 82 and 83

Due to the liquidation of NYC, no scale factor for MIA is applied, all votes are equally counted.

This CCIP also uses the updating voting methods used with CCIP-020 without the 25% participation threshold per city.
The votes are tallied per city and available in read-only functions for information purpose only.

The proposal is activated after a voting period of 2 weeks

- if the votes for the proposal are greater than the votes against the proposal.

## Reference Implementations

- The proposal contract is available here: https://github.com/citycoins/protocol/pull/83
