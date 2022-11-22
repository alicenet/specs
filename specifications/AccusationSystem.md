---
title: Proposal for an Accusation System
author: ricardopinto
discussion: https://github.com/alicenet/specs/discussions/
team: Backend
category: Development
status: Draft
related: [Accusations manager](https://github.com/alicenet/alicenet/pull/37) , [Multiple Proposal Accusation](https://github.com/alicenet/alicenet/pull/239) , [Invalid utxo consumption](https://github.com/alicenet/alicenet/pull/255), [Accusation manager can be enabled/disabled in configuration file](https://github.com/alicenet/alicenet/pull/324)
created: 2022-11-21
---

## Introduction

AliceNet is a Proof-of-Stake blockchain that allows anyone to become a validator and participate in consensus. Ensure correct behavior and security in AliceNet is of utmost importance and hence the need to create such system.

#### Summary

This specification document captures the current requirements and state of development for the accusation system.

#### Context

The accusation system monitors proposals from the consensus algorithm and initiates an accusation against the malicious validator to ensure long term stability and security of the blockchain. Validators participating in consensus, having the accusation system enabled, will automatically verify each proposal for misconduct. When detected, a cryptographic proof will be generated and sent to AliceNet smart contracts on the Ethereum mainnet.

#### Goals

- Automatically detect known conditions of malicious behavior
- Automatically evict/slash malicious actors using cryptographic proofs
- Reward accusers for correctly accusing bad actors and contributing to a secure blockchain
- The accusation system must be lightweight, resource efficient, and scale with the number of validators
- The accusation system can be enabled/disabled in a configuration file
- The accusation system runs concurrently with the consensus algorithm
- When a malicious validator is correctly accused, all validators must ignore/reject every proposal from it
- Implement the first algorithm to [detect multiple proposals](https://github.com/alicenet/alicenet/pull/239) from a validator, aka forking

#### Non Goals

- Verify integrity of each proposal because that's already done on the core consensus

#### Assumptions

- For the accusation system to work, validators must be engaging in the consensus protocol and sealing blocks
- Only a validator can accuse another validator
- At the moment, wrongfully accusing a validator yields no penalty to the accuser
- For any given proposal, only one accusation is ever detected and the remaining ones are ignored because this validator will already be evicted

## Specification

#### Overview

The current design attaches the `AccusationManager` instance to the consensus engine loop, where it is responsible for polling consensus round states and concurrently verifying each proposal through a pipeline of detection algorithms. These detection algorithms are tailored to identify specific malicious acts and are responsible for generating cryptographic proofs which are then sent to AliceNet smart contracts in the Ethereum mainnet.

#### Data

Whenever the accusation system detects a malicous intent, it will generate all the necessary proof data for sending to the smart contracts for validation. Even though each accusation detection algorithm required different proof data, a generic structure is in place to accomodate the many types of accusations. This accusation proof data is marshalled and stored in the consensus DB with the [prefix](https://github.com/alicenet/alicenet/pull/37/files#diff-c3ea7eae92616737c92b0f0d4fe966de69fd9c40fc2f4f9d0aac6a504ce16554R138) `0xa7`. For the purpose of this spec no details on concrete detection algorithms and their proof data is provided, and instead each accusation algorithm should have its own Spec document.

Whenever a validator is accused, validators keep track of this be storing evicted validators in the consensus DB with the [prefix](https://github.com/alicenet/alicenet/pull/37/files#diff-c3ea7eae92616737c92b0f0d4fe966de69fd9c40fc2f4f9d0aac6a504ce16554R142) `0xa8`.

#### Logic
<!--- APIs / Pseudocode / Flowcharts / Conditions / Limitations -->

API:
- DB.GetEvictedValidators()
- DB.IsValidatorEvicted()
- DB.SetAccusation()
- DB.RemoveAccusation()
- DB.GetAllAccusations()

Pseudocode:

Flowcharts:

Conditions:
- The accusation system only runs when consensus is in sync

Limitations:
- Detecting some malicious acts is in some cases unpractical at this level given how malicious proposal data is verified and discarded in others parts of the codebase.
- Reproducing and testing some accusation scenarios is not a trivial task.

#### Presentation

N/A

#### Testing

Reproducing and testing some accusation scenarios is not a trivial task.

#### Security / Risks

TBD

## Further Considerations

#### Alternative Solutions

N/A

#### Timeline

N/A

#### Prioritization

N/A

#### Dependencies

N/A

#### Open Questions

N/A
