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

For Context please refer to [Native ERC Bridge Pool Base](https://github.com/alicenet/specs/pull/16).

#### Goals

Allow for ERC721 assets to be transferred from Ethereum to Alicenet and back.

#### Non Goals

Do not enable ERC721 token bridging between Alicenet and other L2 chains

#### Assumptions

Smart contract functions will be called by user through Alicenet Wallet

## Specification

#### Overview

For Overview please refer to [Native ERC Bridge Pool Base](https://github.com/alicenet/specs/pull/16).

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

For deposit parameters plase refer to [Native ERC Bridge Pool Base](https://github.com/alicenet/specs/pull/16).

Upon this call, the contract should perform the following operations:
1. Call deposit function in NativeERCBridgePoolBase.sol through [super.deposit()](https://github.com/alicenet/specs/pull/16), this function is right now just a placeholder for future deposit pre-actions.
2. Transfer tokens from the caller to smart contract and hold them indefinitely.

##### Withdraw function

For withdraw parameters plase refer to [Native ERC Bridge Pool Base](https://github.com/alicenet/specs/pull/16).

Upon this call, the contract should perform the following operations:
1. Call withdraw function in [NativeERCBridgePoolBase.sol](https://github.com/alicenet/specs/pull/16) through super.withdraw(), upon proof verification this call will return token amount to transfer.
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

For Security / Risks please refer to [Native ERC Bridge Pool Base](https://github.com/alicenet/specs/pull/16).

## Further Considerations

#### Timeline

Should be released on Q1/2023

#### Prioritization

Low / Medium priority.

#### Alternative Solutions

N/A

#### Dependencies

[Native ERC Bridge Pool Base](https://github.com/alicenet/specs/pull/16), Bridge Pool Factory and Central BridgeRouter

#### Open Questions

Can this be implemented through ERC1155?