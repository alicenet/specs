---
title: Native Bridge Pool ERC721
author: @gusjavaz
discussion: https://github.com/alicenet/specs/issues/9
team: Backend
category: Development
status: Draft
related: https://github.com/alicenet/alicenet/issues/423
created: 2022-11-22
---

## Introduction

#### Summary

This specification allows ERC721 tokens to be transferred from Ethereum Mainnet into Alicenet and back.

#### Context

Bridge Pools allow transfers of assets between chains, Alicenet Native Bridge Pools allow transfers between Ethereum Mainnet and Alicenet both ways.
The main reasons for bridging assets into Alicenet is that transactions in Alicenet are:
* Cheaper than Ethereum
* Faster than Ethereum
* As safe as in Ethereum

At high level Bridge Pool contracts operate as following:
* Upon depositing tokens -> Transfer the tokens from the owner to the Bridge Pool contract and after validation trigger the minting of corresponding tokens on L2 chain.
* Upon withdrawing tokens -> Trigger the burning of the L2 chain corresponding tokens and after validation transfer the tokens from the Bridge Pool Contract to the owner.

#### Goals

Allow for ERC721 assets to be transferred from Ethereum to Alicenet and back.

#### Non Goals

Do not enable ERC721 token bridging between Alicenet and other L2 chains

#### Assumptions

Smart contract functions will be called by user through Alicenet Wallet

## Specification

#### Overview

To enable bridging for a specific ERC721 token the following tasks must be performed:
* By administrator -> Native Bridge Pool implementations with specific versions should be previously created (EX: NativeERC721BridgePoolV1.sol, NativeERC721BridgePoolV2.sol ), these implementations must inherit from [NativeERCBridgePoolBase.sol](https://github.com/alicenet/specs/pull/16).
* By user -> The selected implementation version should be deployed by the Bridge Pool Factory and initialized with the specified ERC721 token contract address.
* By Central Bridge Router contract -> Posteriorly route calls for depositing/withdrawing to the correct implementation. 

#### Data

N/A

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

Through this operation Native Bridge Pool contract receives ERC721 contract's address that represents the token to bridge, this contract address will support transfer operations

##### Deposit function

Deposit function should be called only through Bridge Router with the following parameters:
1. Sender address -> EOA of the user who will be sending tokens (not msg.sender)
2. Amount -> Number of tokens to deposit (uint256)

Upon this call the contract should perform the following operations:
1. Call deposit function in NativeERCBridgePoolBase.sol through [super.deposit()](https://github.com/alicenet/specs/pull/16), this function is right now just a placeholder for future deposit pre-actions.
2. Transfer tokens from the caller to smart contract and hold them indefinitely.

##### Withdraw function

Withdraw function should be called only through BridgeRouter.sol with the following parameters:
1. Receiver address -> EOA of the user who will be receiving tokens (not msg.sender)
2. Burned UTXO -> an Alicenet preImage of burned UTXO
3. Proof of inclusion in StateRoot: Proof of inclusion of UTXO in the stateTrie
4. Proof of inclusion in TXRoot: Proof of inclusion of the transaction included in the txRoot trie.
5. Proof of inclusion in TXHash: Proof of inclusion of txOut in the txHash trie
6. Proof of inclusion in HeaderRoot: Proof of inclusion of the block header in the header trie

Upon this call the contract should perform the following operations:
1. Call withdraw function in NativeERCBridgePoolBase.sol through  [super.withdraw()](https://github.com/alicenet/specs/pull/16), upon proof verification this call will return token amount to transfer.
2. Transfer returned token amount from the ERC721 contract to the specified receiver.

#### Error Handling

The errors thrown by this contract are specified in [NativeERCBridgePoolBase.sol](https://github.com/alicenet/specs/pull/16) parent contract. 

#### Testing

Native Bridge Pool ERC721 contracts should pass the following tests:
* Should deposit if called from Bridge Router
* Should fail to deposit if not called from Bridge Router
* Should make a withdraw for amount specified on informed burned UTXO upon proof verification
* Should fail to make a withdraw if not called from Bridge Router
* Should not make a withdraw on an already withdrawn UTXO upon proofs verification
* Should not make a withdraw for amount specified on informed burned UTXO with wrong Merkle proof
* Should not make a withdraw if sender does not match UTXO owner
* Should not make a withdraw if chainId in UTXO does not match chainId in snapshot's claims
* Should not make a withdraw if UTXOID in UTXO does not match UTXOID in proof
* Should not call a withdraw if state key does not match txhash key

#### Presentation

N/A

#### Security / Risks

The following risks should be taken in account on withdraw function:
* Calling with fake Merkle Proofs that satisfies proof verification.
* Forcing of getLatestSnapshot() on Snapshots.sol to provide fake Block Claims that satisfies proof verification.

## Further Considerations

#### Timeline

Should be released on Q1/2023

#### Prioritization

Low / Medium priority.

#### Alternative Solutions

N/A

#### Dependencies

NativeERCBridgePoolBase, Bridge Pool Factory and Central BridgeRouter

#### Open Questions

Can this be implemented through ERC1155?