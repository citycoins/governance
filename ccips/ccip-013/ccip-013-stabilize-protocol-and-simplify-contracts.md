# CCIP-013

## Preamble

| CCIP Number   | 013                                       |
| ------------- | ----------------------------------------- |
| Title         | Stabilize Protocol and Simplify Contracts |
| Author(s)     | Jason Schrader jason@joinfreehold.com     |
| Consideration | Economic, Governance, Technical           |
| Type          | Standard                                  |
| Status        | Draft                                     |
| Created       | 2022-08-15                                |
| License       | BSD-2-Clause                              |
| Replaces      | TBD                                       |

## Introduction

This proposal implements the second of two changes to help stabilize the CityCoins protocol design, and allow for future development, experimentation, and growth.

Please see CCIP-012[^1] for the first part of the proposal.

This proposal is designed to work in concert with Phase 1 and 2 of CCIP-012[^1], and consists of the final third phase:

- Phase 3: stabilize and simplify the protocol

## Specification

### Phase 3: Stabilize and Upgrade Protocol Contracts

This phase implements two main changes to help simplify the CityCoins protocol design, and bring ownership and execution of CityCoin contract updates into the DAO setup in Phase 2.

1. Stabilize the CityCoins Protocol
2. Update Registration, Mining and Stacking flows

#### Simplify the CityCoins Protocol

The original design allowed each city to grow and change independent of the core protocol specification, with the intent that each city would eventually manage their local deployment.

Under the current protocol, each city requires three smart contracts that cover three main areas:

- Auth: initialization and protected functions
- Core: user registration, activation, mining and stacking
- Token: SIP-010 functions, send-many support

This design choice prioritized customization at the city level but also led to some unforeseen challenges, including:

- inconsistencies between contract code being developed vs already deployed (TODO: link to diff between v1 MIA and v1.1 NYC)
- no universal identification for users, instead one user ID per core contract (TODO: show example of v1 vs v2 MIA and NYC IDs)
- mining and stacking data does not migrate after a core contract upgrade
- no universal registry for contracts, all upgrades are tracked on a city by city level
- an unsustainable unit testing model that grows exponentially with new cities (TODO: example of 900+ tests run on GitHub)

In order to form a more cohesive protocol and consistent experience across all CityCoins, and to reduce the amount of overhead for protocol upgrades and maintenance, this could be simplified into a structure that takes advantage of the initial DAO created in Phase 2.

Using this DAO structure, proposals would be created and executed that:

- create a general registration contract for all CityCoins that would track:
  - user registration for all CityCoins
  - available cities and status (enabled/disabled)
  - treasury addresses for each city
- create a general mining contract for all CityCoins that would track:
  - mining stats per block, per city
  - mining stats per user, per block, per city
  - mining exchange rate for a city
  - mints new CityCoins to block winners
- create a general stacking contract for all CityCoins that would track:
  - stacking stats per cycle, per city
  - stacking stats per user, per cycle, per city
  - stacking total value locked per city
- adds MIA and NYC as activated cities under the new protocol

Any new cities activated following these changes would only require a token contract and the necessary treasury contracts, all of which could be decided upon and instantiated through DAO proposals.

#### Update Mining and Stacking flows

In addition to the overall protocol changes above, this phase implements a change to the value flows in mining and stacking, such that:

- 100% of STX spent mining CityCoins is transferred to the city’s treasury
- the STX within the city’s treasury are stacked for xBTC
- a portion of xBTC rewards can be claimed by CityCoin Stackers who participate by locking their CityCoins for a reward cycle

![CityCoins Ecosystem Structure Proposal](citycoins-ecosystem-structure-proposal.png)

This change would make it so that CityCoin stacking cycles become dependent on Stacks stacking cycles.
