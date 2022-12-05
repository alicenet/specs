---
title: Proposal for an Accusation against Invalid UTXO Consumption
author: ricardopinto
discussion: https://github.com/alicenet/specs/discussions/
team: Backend
category: Development
status: Draft
related: [Accusations manager](https://github.com/alicenet/alicenet/pull/37)
created: 2022-12-02
---

## Introduction

The accusation system monitors and acts on malicious validator behaviors. One class of such behaviors is consumption of invalid UTXOs that could allow a validator to propose a block where a transaction would use an invalid or non-existant UTXO as its input, meaning it would be trying to spend funds out of thin air.

#### Summary

This specification document captures the current requirements and state of development for an accusation detection algorithm against invalid UTXO consumption.

#### Context

AliceNet is a UTXO based blockchain and as such it requires a transaction to reference past UTXOs as their source. One potential issue that can arise is when a trasaction tries to reference UTXOs that don't exist, or have already been spent before, and even though there are many validations in place for such a transaction to be rejected before even being considered for inclusion in a block proposal, it is still possible that a malicious validator can try to propose such a transation. This spec document concerns only to the scenarios where UTXOs are invalid or non-existent, and does not concern to the cases where a UTXO has already been consumed.

#### Goals

- Automatically detect proposals containing transactions referencing invalid or non-existent UTXOs.
- Automatically evict/slash malicious actors using cryptographic proofs.
- Reward accusers for correctly accusing bad actors and contributing to a secure blockchain.

#### Non Goals

- It is not a goal to verify integrity of each proposal because that's already done on the core consensus and also when unmarshalling proposal data at the peer-to-peer layer.
- It is not a goal to assert if the UTXO has already been consumed

#### Assumptions

- For this accusation detection algorithm, it is assumed that there's a system responsible for managing accusations and their runtime.

## Specification

#### Overview

The current design integrates the detection algorithm to the [AccusationManager](https://github.com/alicenet/alicenet/pull/37/files#diff-98dadcc1c48e83179e98504fae590b5d3b4441b8c1f330add60740079ac31549R53), where it is responsible for detecting the conditions for an invalid UTXO consumption scenario.

#### Data

A successful invalid UTXO accusation requires cryptographic proofs to be submitted to the smart contracts. This proof data must contain following information:
- Proposal PClaims data
- Proposal PClaims signature
- Proposal BClaims data
- Proposal BClaims group signature
- The `TXInPreImage` consuming the invalid UTXO
- Merkle proofs:
  - Proof against StateRoot: proof of inclusion or exclusion of the deposit or UTXO in the stateTrie
  - Proof of inclusion in TXRoot: proof of inclusion of the transaction that included the invalid input in the txRoot trie.
  - Proof of inclusion in TXHash: proof of inclusion of the invalid input (txIn) in the txHash trie (transaction tested against the TxRoot).

This data alone is enough to verify the accusation proof integrity by the smart contracts.

#### Logic
<!--- APIs / Pseudocode / Flowcharts / Conditions / Limitations -->

Conditions:
- This detection logic is active when the accusation system is active


#### Presentation

N/A

#### Testing



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

Accusation System [spec](https://github.com/alicenet/specs/issues/6) and [PR](https://github.com/alicenet/alicenet/pull/37).

#### Open Questions

- Under this accusation, should a validator be `minor` or `major` slashed?
