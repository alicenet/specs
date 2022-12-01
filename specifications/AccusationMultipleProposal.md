---
title: Proposal for an Accusation against Multiple Proposal
author: ricardopinto
discussion: https://github.com/alicenet/specs/discussions/
team: Backend
category: Development
status: Draft
related: [Accusations manager](https://github.com/alicenet/alicenet/pull/37) , [Multiple Proposal Accusation](https://github.com/alicenet/alicenet/pull/239) , [Invalid utxo consumption](https://github.com/alicenet/alicenet/pull/255), [Accusation manager can be enabled/disabled in configuration file](https://github.com/alicenet/alicenet/pull/324)
created: 2022-12-01
---

## Introduction

The accusation system monitors and acts on malicious validator behaviors. One class of such behaviors is multiple proposals that could allow a validator to fork the AliceNet chain.

#### Summary

This specification document captures the current requirements and state of development for an accusation detection algorithm against multiple proposals.

#### Context

Validators submit block proposals on each consensus height and round, and these proposals are validated by other validators. In some cases it is possible for the same validator to submit two different proposals and this could lead to a fork in AliceNet. Such a scenario can happen when a validator has more than one validator node software running.

#### Goals

- Automatically detect multiple proposals out of a validator.
- Automatically evict/slash malicious actors using cryptographic proofs.
- Reward accusers for correctly accusing bad actors and contributing to a secure blockchain.

#### Non Goals

- It is not a goal to verify integrity of each proposal because that's already done on the core consensus and also when unmarshalling proposal data at the peer-to-peer layer.

#### Assumptions

- For this accusation detection algorithm, it is assumed that there's a system responsible for managing accusations and their runtime.

## Specification

#### Overview

The current design integrates the detection algorithm to the [AccusationManager](https://github.com/alicenet/alicenet/pull/37/files#diff-98dadcc1c48e83179e98504fae590b5d3b4441b8c1f330add60740079ac31549R53), where it is responsible for detecting the conditions for a multiple proposal scenario.

#### Data

A successful multiple proposal accusation requires cryptographic proofs to be submitted to the smart contracts. This proof data requires the following information:
- Proposal signature
- Proposal PClaims data
- Conflicting proposal signature
- Conflicting proposal PClaims data
- The proposer address

This data alone is enough to verify the accusation proof integrity by the smart contracts.

#### Logic
<!--- APIs / Pseudocode / Flowcharts / Conditions / Limitations -->

Conditions:
- This detection logic is active when the accusation system is active


Pseudocode:

To detect multiple proposal scenarios, several conditions must be met by the algorithm:

1. There are two proposals from the same validator
2. The proposals are different
3. The proposals have different PClaims data
4. The proposer is an actual validator
5. The proposer is the supposed validator to propose at the current height and round
6. The RClaims on both proposals are the same; RClaims are found in Proposal.PClaims.RCert hierarchy
7. The signatures of each proposal must be different, yet valid

#### Presentation

N/A

#### Testing

To reproduce the conditions for this accusation algorithm, a validator can clone/duplicate it's node software and data, and have them both running in parallel. The next step is to create transactions on one of these nodes, through RPC, which will make the RPC receiving node to propose blocks that contain the transactions while the other node software will not, hence resulting in a multiple proposal from this same validator.

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

N/A
