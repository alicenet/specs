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

This data alone is enough for the smart contracts to verify the accusation proof data integrity.

#### Logic
<!--- APIs / Pseudocode / Flowcharts / Conditions / Limitations -->

Conditions:
- This detection logic is active when the accusation system is active

Smart contracts:

The smart contract implementation to validate accusation proofs and evict validators is already in place [here](https://github.com/alicenet/alicenet/pull/37/files#diff-cfd0be5e9ca0938babbbde8460a189be71fd805ba34e0dc53a92b584192164e5).

##### Smart contract:

The [AccusationInvalidTxConsumption](https://github.com/alicenet/alicenet/pull/37/files#diff-cfd0be5e9ca0938babbbde8460a189be71fd805ba34e0dc53a92b584192164e5R19) contract is responsible for receiving and processing the accusation proof data, validating the proofs and evicting the malicious validator from the validator pool.

###### API
```solidity

contract AccusationInvalidTxConsumption {

  /// @notice This function validates an accusation of non-existent utxo consumption, as well as invalid deposit consumption.
  /// @param pClaims_ the PClaims of the accusation
  /// @param pClaimsSig_ the signature of PClaims
  /// @param bClaims_ the BClaims of the accusation
  /// @param bClaimsSigGroup_ the signature group of BClaims
  /// @param txInPreImage_ the TXInPreImage consuming the invalid transaction
  /// @param proofs_ an array of merkle proof structs in the following order:
  ///   proof against StateRoot: Proof of inclusion or exclusion of the deposit or UTXO in the stateTrie
  ///   proof of inclusion in TXRoot: Proof of inclusion of the transaction that included the invalid input in the txRoot trie.
  ///   proof of inclusion in TXHash: Proof of inclusion of the invalid input (txIn) in the txHash trie (transaction tested against the TxRoot).
  /// @return the address of the signer
  function accuseInvalidTransactionConsumption(
        bytes memory pClaims_,
        bytes memory pClaimsSig_,
        bytes memory bClaims_,
        bytes memory bClaimsSigGroup_,
        bytes memory txInPreImage_,
        bytes[3] memory proofs_
  )
  public
  returns (address)

  /// @notice This function tells whether an accusation ID has already been submitted or not.
  /// @param id_ The deterministic accusation ID
  /// @return true if the ID has already been submitted, false otherwise
  function isAccused(bytes32 id_) public view returns (bool)
}
```

###### Smart contract algorithm to successfully validate accusation proofs
1. Ensure previous block is signed by correct group key for current validator set
2. Ensure there are transactions by checking `PClaims.BClaims.TxCount`
3. Ensure height delta is 1
4. Ensure the chain ID is correct for `PClaims.BClaims.ChainID`
5. Recover the malicious signer address using `ecrecover` signature verification of `PClaims`, and ensure the signer is an active validator
6. Ensure ProofInclusionTxRoot against PClaims.BClaims.TxRoot is valid
7. Ensure ProofOfInclusionTxHash against the target hash from ProofInclusionTxRoot is valid
8. Ensure the transaction UTXO ID does not exist
7. Evict the malicious signer from the validator pool


#### Presentation

N/A

#### Testing

Reproducing this scenario on a live network is proving challenging because transactions referencing invalid UTXOs are rejected early in the RPC and peer-to-peer layers when data is checked for integrity. Because of this, the detection algorithm doesn't get to see this transaction and is therefore not able to accuse the malicious validator.

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
