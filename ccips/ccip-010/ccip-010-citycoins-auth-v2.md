# CCIP-010

## Preamble

| CCIP Number   | 010                                   |
| ------------- | ------------------------------------- |
| Title         | CityCoins Auth V2                     |
| Author(s)     | Jason Schrader jason@joinfreehold.com |
| Consideration | Technical                             |
| Type          | Standard                              |
| Status        | Draft                                 |
| Created       | 2022-03-25                            |
| License       | BSD-2-Clause                          |
| Replaces      | CCIP-007                              |

## Introduction

The Stacks blockchain is a Layer 1 blockchain connected to Bitcoin, in which miners spend Bitcoin to bid for and win a fixed amount of Stacks tokens. Stackers have the option to lock up Stacks tokens for a specified amount of time, and in turn, receive a portion of the Bitcoin spent by miners proportionate to the amount Stacked[^1].

Taking this concept a level further, a new token (a "CityCoin") is created on the Stacks blockchain, following the SIP-010 fungible token standard[^2], such that CityCoins can be mined and Stacked per the methods stated above except a portion of the miner's bid would be redirected to the city's wallet overseen by a trusted third party custodian.

CityCoins leverage similar properties from the Proof of Transfer (PoX) consensus mechanism[^3] of the Stacks blockchain, programmed through a smart contract in Clarity.

This proposal describes the protected functions used in the auth contract for a CityCoin.

## Specification

### Protected Functions

The auth contract for a CityCoin contains protected functions that allow the city wallet or a backup list of approvers to authorize:

- an update to the city wallet address, in case the custody address is changed
- an upgrade to the core contract, replacing the current core contract with a new one
- an adjustment to the emissions schedule
- an update to the token URI, which specifies the off-chain location of the CityCoin token data

The city wallet is a 2-of-3 multisignature wallet, and requires 2-of-3 approval before any of the protected functions can be used.

An additional backup 3-of-5 approval method is instantiated among independent entities before any of the protected functions can be used.

### Approval Flow

The auth contract allows the city wallet to call any of the protected functions once the required number of signatures have been collected.

The backup approver list must submit a job through a specific flow designed to allow approvers to:

- create a job for approval with specific parameters
  - city wallet update must have a principal value for `newCityWallet`
  - core contract upgrade must have principal values for `oldContract` and `newContract`
  - replacing approvers must have principal values for `oldApprover` and `newApprover`
- cast votes on a job for approval/disapproval
- reverse votes on a job for approval/disapproval
- execute a specific jobs if the approval threshold is met
  - city wallet update uses `execute-set-city-wallet-job`
  - core contract upgrade uses `execute-upgrade-core-contract-job`
  - replacing approvers uses `execute-replace-approver-job`

### Contract Operation

Within the auth contract are variables to track information across two categories.

Related to core contracts:

- a map of core contracts and their state
- the current active core contract principal
- the city wallet address

Related to jobs and approvals:

- a map of approvers and their state
- the required approvals for jobs
- the ID of the last job created
- job information and status
- votes by approvers for each job
- a list of the approver addresses
- argument details by name and ID

### Core Contract States

Core contracts states are defined below:

| State          | uint | Description                                         |
| -------------- | ---- | --------------------------------------------------- |
| STATE_DEPLOYED | u0   | Core contract deployed but not activated            |
| STATE_ACTIVE   | u1   | Core contract activated, mining/stacking available  |
| STATE_INACTIVE | u2   | Core contract shutdown, mining/stacking unavailable |

### Updating the City Wallet Address

When the `set-city-wallet` function is called, the contract then:

- verifies the caller is the city wallet principal
- verifies the core contract is the active core contract
- sets the city wallet variable in auth and the active core contract

When the `execute-set-city-wallet-job` function is called, the contract then:

- verifies the correct job arguments are found
- verifies the caller is in the approvers map
- verifies the core contract is the active core contract
- sets the city wallet variable in auth and the active core contract
- marks the job as executed
  - verifies the job ID is found
  - verifies the job is active
  - verifies the job has the required approvals
  - verifies the auth contract is marking the job as executed
  - verifies job was not already executed
  - updates the job status to executed

### Upgrading a Core Contract

When the `upgrade-core-contract` function is called, the contract then:

- verifies the old core contract is in the core contract map
- verifies the old and new core contract are not the same
- verifies the caller is the city wallet principal
- updates the core contract states
  - old contract is set to STATE_INACTIVE
  - new contract is set to STATE_DEPLOYED
  - the active contract variable is updated to the new contract principal
- calls the `shutdown-contract` in the old core contract
- sets the city wallet variable in the new core contract

When the `execute-upgrade-core-contract-job` function is called, the contract then:

- verifies the correct job arguments are found
- verifies the old core contract is in the core contract map
- verifies the caller is in the approvers map
- verifies the old and new core contract are not the same
- updates the core contract states
  - old contract is set to STATE_INACTIVE
  - new contract is set to STATE_DEPLOYED
  - the active contract variable is updated to the new contract principal
- calls the `shutdown-contract` in the old core contract
- sets the city wallet variable in the new core contract
- marks the job as executed
  - verifies the job ID is found
  - verifies the job is active
  - verifies the job has the required approvals
  - verifies the auth contract is marking the job as executed
  - verifies job was not already executed
  - updates the job status to executed

### Updating the Emissions Schedule

TODO: add details

### Updating the Token URI

When the `set-token-uri` function is called, the contract then:

- verifies the caller is the city wallet principal
- updates the token URI variable in the token contract

### Replacing an Approver

When the `execute-replace-approver-job` function is called, the contract then:

- verifies the correct job arguments are found
- verifies the caller is in the approvers map
- disable the old approver address
- add and enable the new approver address
- marks the job as executed
  - verifies the job ID is found
  - verifies the job is active
  - verifies the job has the required approvals
  - verifies the auth contract is marking the job as executed
  - verifies job was not already executed
  - updates the job status to executed

## Backwards Compatibility

This CCIP replaces the auth contract in CCIP-007 and is not backwards compatible.

## Activation

TODO: On-chain vote description

## Reference Implementations

TODO: add references

## Footnotes

[^1]: https://stacking.club
[^2]: https://github.com/stacksgov/sips/blob/main/sips/sip-010/sip-010-fungible-token-standard.md
[^3]: https://docs.stacks.co/understand-stacks/proof-of-transfer
[^4]: https://explorer.stacks.co/txid/SP466FNC0P7JWTNM2R9T199QRZN1MYEDTAR0KP27.miamicoin-auth?chain=mainnet
[^5]: https://explorer.stacks.co/txid/SP2H8PY27SEZ03MWRKS5XABZYQN17ETGQS3527SA5.newyorkcitycoin-auth?chain=mainnet
