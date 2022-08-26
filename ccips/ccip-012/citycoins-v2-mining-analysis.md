# CityCoins V2 Mining Analysis

## Summary

This document outlines the analysis of CityCoins V2 mining and stacking activity performed on 2022/08/24.

A set of [Node.js scripts](https://nodejs.dev/en/) were used to gather the data, available in the [analysis sub-folder](./citycoins-v2-mining-analysis).

All of the data is stored in JSON arrays, including:

- `miaTxs.json` - all MiamiCoin v2 transactions
- `nycTxs.json` - all NewYorkCityCoins v2 transactions
- `userdata-mia` - compiled info per address based on transactions for MiamiCoin v2
- `userdata-nyc` - compiled info per address based on transactions for NewYorkCityCoin v2
- `txdata-mia` - all transactions for selected miners of MiamiCoin v2
- `txdata-nyc` - all transactions for selected miners of NewYorkCityCoin v2

The data was then compiled into a spreadsheet, available [in the analysis sub-folder](./citycoins-v2-mining-analysis/citycoins-v2-mining-analysis.ods), which shows the data per CityCoin on each tab.

## Data Collected and Analyzed

The scripts collected the following data from the CityCoins V2 contracts:

- all transactions from `miamicoin-core-v2`
- all transactions from `newyorkcitycoin-core-v2`

Then using that data, the following was calculated per `sender_address`:

- User ID
- Total Blocks Mined
- Total STX Committed
- Total CityCoins Claimed
- Total Stacked All Time
- Total Stacked This Cycle
- Total Stacking Rewards

Following that, of the top miners who did not participate in stacking, the following was gathered and analyzed:

- all transactions from the miners' addresses
- Total Sent to Okcoin, based on known Okcoin address `SP3HXJJMJQ06GNAZ8XWDN1QM48JEDC6PP6W3YZPZJ`
- % of CityCoins Claimed, based on Total CityCoins Claimed
