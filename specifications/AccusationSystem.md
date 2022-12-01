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

AliceNet is a Proof-of-Stake blockchain that allows anyone to become a validator and participate in consensus. Ensuring correct behavior and security in AliceNet is of utmost importance and hence the need to create a system to monitor and act on malicious validators.

#### Summary

This specification document captures the current requirements and state of development for the accusation system.

#### Context

As validators coordinate to seal AliceNet blocks, some might try and introduce malicious proposals that can affect the network integrity and security. Accounting for that, the accusation system monitors proposals from the consensus algorithm and initiates accusations against malicious validators to evict them from the validators pool. Validators participating in consensus, having the accusation system enabled, will automatically verify each proposal for misconduct, and when malicious intent is detected, it will generate a cryptographic proof to sent to AliceNet smart contracts on the Ethereum mainnet.

#### Goals

- Automatically detect known conditions of malicious behavior.
- Automatically evict/slash malicious actors using cryptographic proofs.
- Reward accusers for correctly accusing bad actors and contributing to a secure blockchain.
- The accusation system must be lightweight, resource efficient, and scale with the number of validators.
- The accusation system can be enabled/disabled in a configuration file.
- The detection algorithms run concurrently with the consensus algorithm.
- When a malicious validator is correctly accused, all validators must ignore/reject every proposal from it.
- Implement the first algorithm to [detect multiple proposals](https://github.com/alicenet/alicenet/pull/239) from a validator, also known as forking.

#### Non Goals

- It is not a goal to verify integrity of each proposal because that's already done on the core consensus and also when unmarshalling proposal data at the peer-to-peer layer.

#### Assumptions

- For the accusation system to work, validators must be engaging in the consensus protocol and sealing blocks.
- Only a validator can accuse another validator.
- At the moment, wrongfully accusing a validator yields no penalty to the accuser. In this case the accuser would only lose the ethereum transaction fee.
- For any given proposal, only one accusation should ever be detected and the remaining ones are ignored because this validator will already be evicted.

## Specification

#### Overview

The current design attaches the [AccusationManager](https://github.com/alicenet/alicenet/pull/37/files#diff-98dadcc1c48e83179e98504fae590b5d3b4441b8c1f330add60740079ac31549R53) instance to the [consensus engine loop](https://github.com/alicenet/alicenet/pull/37/files#diff-6d04dd09734c8e9351bc34e40bfd01bb60a487e7653f956aa83cbf8344301a30R536), where it is responsible for polling consensus round states and concurrently verifying each proposal through a pipeline of detection algorithms. These detection algorithms are tailored to identify specific malicious acts and are responsible for generating cryptographic proofs which are then sent to AliceNet smart contracts in the Ethereum mainnet.

#### Data

Whenever the accusation system detects a malicous intent, it will generate all the necessary proof data for sending to the smart contracts for validation. Even though each accusation detection algorithm requires different proof data, a generic structure is in place to accomodate the many types of accusations. This accusation proof data is marshalled and stored in the consensus DB with the [prefix](https://github.com/alicenet/alicenet/pull/37/files#diff-c3ea7eae92616737c92b0f0d4fe966de69fd9c40fc2f4f9d0aac6a504ce16554R138) `0xa7`. For the purpose of this spec no details on concrete detection algorithms and their proof data is provided, and instead each accusation algorithm should have its own Spec document.

Whenever a validator is accused, validators keep track of this be storing evicted validators in the consensus DB with the [prefix](https://github.com/alicenet/alicenet/pull/37/files#diff-c3ea7eae92616737c92b0f0d4fe966de69fd9c40fc2f4f9d0aac6a504ce16554R142) `0xa8`.

Smart contracts in the Ethereum mainnet receive and verify accusation proof data using merkle proofs and signature validations. Examples of existing accusation smart contracts can be found for [multiple proposals](https://github.com/alicenet/alicenet/pull/37/files#diff-c61b5edf4da5e02009378cd2307b91d4c37d46cdcddb04d04d67a019faf5e84dR39), [invalid UTXO consumption](https://github.com/alicenet/alicenet/pull/37/files#diff-cfd0be5e9ca0938babbbde8460a189be71fd805ba34e0dc53a92b584192164e5R123), and [double spending of UTXOs](https://github.com/alicenet/alicenet/pull/37/files#diff-cfd0be5e9ca0938babbbde8460a189be71fd805ba34e0dc53a92b584192164e5R113).

Each accusation has a unique deterministic ID that enables other validators to know whether a specific accusation has taken place or not. This ID is generated using the following data:
- Proposer address
- Chain ID
- AliceNet Height
- AliceNet Round
- Type of accusation, here represented as a keccak256 of the accusation name

#### Logic
<!--- APIs / Pseudocode / Flowcharts / Conditions / Limitations -->

Requirements:
- The accusation manager must poll and analyze round state data at least twice as fast as the consensus loop

API:
- `Manager.Poll()` - the accusation manager struct exposes a method to poll the database for round states. This method is called by the engine loop.
- `Database.GetEvictedValidatorsByGroupKey()`
- `Database.SetEvictedValidator()`
- `Database.IsValidatorEvicted()`
- `Database.DeleteAllEvictedValidators()`
- `Database.SetAccusationRaw()`
- `Database.GetAccusationRaw()`
- `Database.GetAccusations()`
- `Database.DeleteAccusation()`

Conditions:
- The accusation system only runs when consensus is in sync.

Limitations:
- Detecting some malicious acts is in some cases unpractical at this level given how malicious proposal data is verified and discarded in others parts of the codebase, without ever reaching the accusation system and the detection algorithms.
- Reproducing and testing some accusation scenarios is not a trivial task.

Pseudocode:
1. The engine synchronizer loop calls the manager's `Poll()` method to trigger the accusation system
2. The accusation manager polls the database for the current round states
3. For each round state, it checks if it was processed before and if not, processes it through a pipeline of detection algorithms
4. If no accusation was found, it continues looking into other round states
5. If an accusation is found, the detection algorithm returns an accusation submission task ready to be executed, which is then sent to the Task Manager
6. Once the accusation is submitted and the validator is evicted, the task finishes and the accusation manager clears up the database

Deterministic accusation ID:

To determine an accusation ID, a keccak256 hash is calculated on the data like so:
```
var preSalt []byte = keccak256([]byte("AccusationMultipleProposal"))

var id []byte = keccak256(
  proposerAddress,
  chainID,
  height,
  round,
  preSalt,
)
```

#### Presentation

N/A

#### Testing

Reproducing and testing some accusation scenarios is not trivial, hence an extended period of testing is needed to ensure correctness and stability.

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

- Accusation IDs are stored independently per accusation class. Should we aggregate all the IDs in a single place for ease of use?
