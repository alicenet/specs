---
title: Native Bridge Pool ERC20
author: @gusjavaz
discussion: alicenet/issues/404
team: Backend
category: Development
status: Draft
related: alicenet/issues/423
created: 2022-11-15
---

## Introduction

#### Summary

This allows ERC20 tokens to be transferred from Ethereum mainnet into Alicenet and back.

#### Context

Bridge Pools allow transfers of assets from one chain to another.
Alicenet Native Bridge Pools allow transfers between Ethereum and Alicenet both ways.
The main reasons for bridging assets into Alicenet is that transfers in Alicenet are:
* cheaper than Ethereum
* faster than Ethereum
* as safe as in Ethereum

#### Goals

Allow for ERC20 assets to be transferred from Ethereum to Alicenet and back.

#### Non Goals

Do not enable transfers between Alicenet and other L2 chains

#### Assumptions

Smart contract functions will be called by user through Alicenet Wallet

## Specification

#### Overview

To enable bridging for a specific ERC20 token, a particular Native Bridge Pool Implementation smart contract should be deployed for the specific ERC20 contract.
These implementations will support multiple version and expose the following operations:
* Initialize 
* Deposit
* Withdraw
These implementations will be deployed by Bridge Pool Factory contract by selecting a specific version and will be initialized for ERC contract address.
A central Bridge Router contract will route calls for depositing/withdrawing to the correct implementation. 

#### Data

N/A

#### Logic

The implementation of Bridge Pool contract should inherit from BridgePoolBase contract and hence implement the following interface:
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
1. Sender address -> EOA
2. Amount -> uint256

Upon this call the contract should perform the following operations:
1. Call deposit function in BridgePoolBase through super.deposit() this function is right now just a placeholder for future deposit pre-actions.
2. Transfer tokens from the caller to smart contract and hold them indefinitely.
3. Trigger in Alicenet the minting of deposited tokens (UTXO creation).

##### Withdraw function

Withdraw function should be called only from Bridge Router with the following parameters:
1. Receiver address -> EOA
2. Burned UTXO -> an Alicenet preImage of burned UTXO
3. Proof of inclusion in StateRoot: Proof of inclusion of UTXO in the stateTrie
4. Proof of inclusion in TXRoot: Proof of inclusion of the transaction included in the txRoot trie.
5. Proof of inclusion in TXHash: Proof of inclusion of txOut in the txHash trie
6. Proof of inclusion in HeaderRoot: Proof of inclusion of the block header in the header trie

Upon this call the contract should perform the following operations:
1. Trigger in Alicenet the burning of withdrawn tokens.
2. Call withdraw function in BridgePoolBase through super.withdraw(), this will execute withdraw pre-actions that involves validation that tokens have been effectively burned by checking the burned UTXO and the merkle proofs that verify that it was burned on the past blocks on Alicenet chain.
3. Transfer withdrawn tokens from the contract to the caller.
4. Register UTXO as already consumed

###### UTXO structure

This is an Alicenet Virtual Store structure:
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
A specific Alicenet structure with additional fields like ERC20 contract address and multiple values should be created for bridging.
For VSPreImage parsing VSPreImageParserLibrary can be used.

###### Proofs validation

For proof validation these libraries can be used:
1. MerkleProofParserLibrary -> Parse proof bytes into a proof structure.
2. MerkleProofLibrary -> Verify inclusion of proof on block claims.

Validation of the Merkle Proof against involves the following steps:
1. Obtain last block claims from Snapshots contract.
2. Validate proofInclusionHeaderRoot against block claims headerRoot.
3. Validate proofInclusionTxRoot against block claims txRoot.
4. Validate proofOfInclusionTxHash against the target hash from proofInclusionTxRoot.
5. Validate proofOfInclusionTxHash against block claims stateRoot.

#### Error Handling

The smart contract should handle the following errors:
1. OnlyBridgePool -> The caller is not Bridge Pool.
2. ChainIdDoesNotMatch -> Chain ID in UTXO is different than chain ID in block claims.
3. UTXODoesnotMatch -> key in proof of inclusion for state root is different than key in proof of inclusion for TxHash.
4. UTXOAlreadyWithdrawn -> UTXO has already been used.
5. UTXOAccountDoesNotMatchReceiver -> The owner of burned UTXO is not the caller.
6. MerkleProofKeyDoesNotMatchUTXOID -> The UTXOID is not on proofs.

#### Testing

Native Bridge Pool ERC20 contracts should pass the following tests:
* Should deposit if called from Bridge Router
* Should fail to deposit if not called from Bridge Router
* Should make a withdraw for amount specified on informed burned UTXO upon proof verification
* Should fail to make a withdraw if not called from Bridge Router
* Should not make a withdraw on an already withdrawn UTXO upon proofs verification
* Should not make a withdraw for amount specified on informed burned UTXO with wrong merkle proof
* Should not make a withdraw if sender does not match UTXO owner
* Should not make a withdraw if chainId in UTXO does not match chainId in snapshot's claims
* Should not make a withdraw if UTXOID in UTXO does not match UTXOID in proof
* Should not call a withdraw if state key does not match txhash key

#### Presentation

N/A

#### Security / Risks

N/A

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

Can this be implemented with ERC1155?