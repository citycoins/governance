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

The current stacking contract `ccd007-citycoin-stacking` has a bug that incorrectly returns 0 STX rewards for a cycle if the following conditions are true:

- the cycle has just ended
- the cycle is not paid by the pool operator
- the user has CityCoins to be returned

When the above conditions are met, the contract does not emit an error and instead calculates the STX reward as 0.

This CCIP will analyze any incorrectly assigned rewards since stacking started in this contract at cycle 54, then transfer the correct STX reward to each affected user.

## Specification

### Research

In order to analyze what users were affected, we need to determine the window in Bitcoin/Stacks block heights that the bug was active in each cycle, then analyze the related transactions to determine who claimed during this period.

For the cycle payouts, we need to look at transactions sent to `ccd011-stacking-payouts` calling the functions `send-stacking-reward-mia` and `send-stacking-reward-nyc`.

For the claims, we need to look at transactions sent to `ccd007-citycoin-stacking` calling the function `claim-stacking-reward` within that window.

### Cycle Data

| Stacking Cycle | BTC Start Block | BTC End Block | STX Start Block | STX End Block | MIA Payout Block | NYC Payout Block |
| -------------- | --------------- | ------------- | --------------- | ------------- | ---------------- | ---------------- |
| 54             | 779450          | 781549        |                 |               |                  |                  |
| 55             | 781550          | 783649        |                 |               |                  |                  |
| 56             | 783650          | 785749        |                 |               |                  |                  |
| 57             | 785750          | 787849        |                 |               |                  |                  |
| 58             | 787850          | 789949        |                 |               |                  |                  |
| 59             | 789950          | 792049        |                 |               |                  |                  |
| 60             | 792050          | 794149        |                 |               |                  |                  |
| 61             | 794150          | 796249        |                 |               |                  |                  |
| 62             | 796250          | 798349        |                 |               |                  |                  |
| 63             | 798350          | 800449        |                 |               |                  |                  |
| 64             | 800450          | 802549        |                 |               |                  |                  |
| 65             | 802550          | 804649        |                 |               |                  |                  |
| 66             | 804650          | 806749        |                 |               |                  |                  |
| 67             | 806750          | 808849        |                 |               |                  |                  |
| 68             | 808850          | 810949        |                 |               |                  |                  |
| 69             | 810950          | 813049        |                 |               |                  |                  |
| 70             | 813050          | 815149        |                 |               |                  |                  |
| 71             | 815150          | 817249        |                 |               |                  |                  |

### Analysis

TBD

## Backwards Compatibility

None, this is a one-time correction.

## Activation

This CCIP will be activated based on the successful vote and activation for CCIP-020.

## Reference Implementations

- TBD

## Footnotes

- TBD
