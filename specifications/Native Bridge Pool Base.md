---
title: Native Bridge Pool ERC20
author: @gusjavaz
discussion: alicenet/issues/423
team: Backend
category: Development
status: Draft
related: alicenet/issues/418
created: 2022-11-18
---

## Introduction

#### Summary

This specification describes the Smart Contracts that allow ERC tokens to be transferred from Ethereum Mainnet into Alicenet and vice versa.

#### Context

Bridge Pools allow transfers of assets between chains, Alicenet Native Bridge Pools allow transfers between Ethereum Mainnet and Alicenet both ways.
The main reasons for bridging assets into Alicenet is that transfers in Alicenet are:
* Cheaper than Ethereum
* Faster than Ethereum
* As safe as in Ethereum

#### Goals

Provide an abstract smart contract (BridgePoolBase.sol) with common required actions for Native Bridge Pools implementations in order to allow them to transfer assets from Ethereum to Alicenet and vice versa.

#### Non Goals

Do not perform transfers, only provide common required actions accessible with super operator.

#### Assumptions

Smart contract functions will be called by Native Bridge Pool ERC implementations through Central Bridge Router

## Specification

#### Overview

To enable bridging for a specific ERC20 token, a particular Native Bridge Pool Implementation smart contract should be deployed for the specific ERC contract.

These implementations will inherit from BridgePoolBase.sol hence exposing the following operations:
* Initialize -> Restricted only from BridgePoolFactory
* Deposit -> Restricted only from BridgeRouter
* Withdraw -> Restricted only from BridgeRouter

These implementations will be deployed by BridgePoolFactory.sol contract through the selection of a specific Bridge Pool Implementation version and will be initialized for the specified ERC contract address.
A central Bridge Router contract will route calls for depositing/withdrawing to the correct implementation. 

#### Data

##### UTXO structure

This is an Alicenet Value Store UTXO structure:
```solidity
    struct VSPreImage {
        uint32 txOutIdx;
        uint32 chainId;
        uint256 value;
        uint8 valueStoreSVA;
        uint8 curveSecp256k1;
        address account;
        bytes32 txHash;
    }
```
A specific Alicenet structure should be created for bridging with additional fields like ERC contract address and multiple values.
For VSPreImage parsing a VSPreImageParserLibrary.sol should be defined.

#### Logic

The implementations of Native Bridge Pool contract should inherit from NativeBridgePoolBase.sol contract, hence implementing the following interface:
```solidity
interface IBridgePool {
    function initialize(address ercContract_) external;

    function deposit(address msgSender, bytes calldata depositParameters) external;

    function withdraw(bytes memory _txInPreImage, bytes[4] memory _proofs)
        external
        returns (address account, uint256 value);
}
```

##### Initialize function

Through this operation Native Bridge Pool contract receives ERC20 contract's address that represents the token to bridge, this contract address will support transfer operations

##### Deposit function

Deposit function should be called only from Bridge Router with the following parameters:
1. Sender address -> EOA of the user who will be sending tokens (not msg.sender)
2. Amount -> Number of tokens to deposit (uint256)

Upon this call the contract should perform the following operations:
1. None -> this function is right now just a placeholder for future deposit pre-actions.

##### Withdraw function

Withdraw function should be called only from BridgeRouter.sol with the following parameters:
1. Receiver address -> EOA of the user who will be receiving tokens (not msg.sender)
2. Burned UTXO -> an Alicenet preImage of burned UTXO
3. Proof of inclusion in StateRoot: Proof of inclusion of UTXO in the stateTrie
4. Proof of inclusion in TXRoot: Proof of inclusion of the transaction included in the txRoot trie.
5. Proof of inclusion in TXHash: Proof of inclusion of txOut in the txHash trie
6. Proof of inclusion in HeaderRoot: Proof of inclusion of the block header in the header trie

Upon this call the contract should perform the following operations:
1. Obtain last block claims from Snapshots contract.
2. Validate proofInclusionHeaderRoot against block claims headerRoot.
3. Validate proofInclusionTxRoot against block claims txRoot.
4. Validate proofOfInclusionTxHash against the target hash from proofInclusionTxRoot.
5. Validate proofOfInclusionTxHash against block claims stateRoot.
6. Return the actual amount of tokens informed on UTXO for token transfer.


###### Merkle Proofs Validation

The following libraries can be used for proof validation:
1. MerkleProofParserLibrary.sol -> Parse proof bytes into a proof structure.
2. MerkleProofLibrary.sol -> Verify inclusion of proof on block claims.


#### Error Handling

The smart contract should handle the following errors:
1. OnlyBridgePool -> The caller is not Bridge Pool.
2. ChainIdDoesNotMatch -> Chain ID in UTXO is different than chain ID in block claims.
3. UTXODoesnotMatch -> Key in proof of inclusion for state root is different than Key in proof of inclusion for TxHash.
4. UTXOAlreadyWithdrawn -> UTXO has already been consumed.
5. UTXOAccountDoesNotMatchReceiver -> The owner of burned UTXO is not the receiver in call.
6. MerkleProofKeyDoesNotMatchUTXOID -> The specified UTXOID is not included on proofs.

#### Testing

Native Bridge Pool ERC20 contracts should pass the following tests:
* Should deposit if called from Bridge Router
* Should fail to deposit if not called from Bridge Router
* Should make a withdraw for amount specified on informed burned UTXO upon proof verification
* Should fail to make a withdraw if not called from Bridge Router
* Should not make a withdraw on an already withdrawn UTXO upon proofs verification
* Should not make a withdraw for amount specified on informed burned UTXO with wrong Merkle proof
* Should not make a withdraw if receiver does not match UTXO owner
* Should not make a withdraw if chainId in UTXO does not match chainId in snapshot's claims
* Should not make a withdraw if UTXOID in UTXO does not match UTXOID in proof
* Should not call a withdraw if state key does not match txhash key

#### Presentation

N/A

#### Security / Risks

The following risks should be taken in account on withdraw function:
* Call with fake Merkle Proofs that satisfies proof verification.
* Force of getLatestSnapshot() on Snapshots.sol to provide fake Block Claims that satisfies proof verification.

## Further Considerations

#### Timeline

Should be released on Q1/2023

#### Prioritization

Low / Medium priority.

#### Alternative Solutions

N/A

#### Dependencies

Bridge Pool Factory and Central BridgeRouter

#### Open Questions

N/A