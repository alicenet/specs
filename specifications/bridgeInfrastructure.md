---
title: AliceNet Bridge
author: @z-j-lin
discussion: <!--- Discussion URL -->
team: Backend
category: Development
status: Draft
related: <!-- Issue or Spec [URL] -->
created: 2022-11-10
---

## Introduction

#### Summary

A set of smart contracts that allow users to pay BToken for transfering ERC20, ERC721, and ERC1155 tokens into and out of alicenet from ethereum

#### Context

The purpose of layer 2 solutions is to increase high transaction through put

#### Goals

Provide users an avenue to transfer tokens on to our layer 2 chain where gas fees are lower and token mobility is higher

#### Non Goals

<!--- What is not to be included with this -->

#### Assumptions

We only charge btoken deposit fees on ethereum for native tokens going into alicnet. All fees for tokens exiting out of alicenet will be collected on the alicenet chain.

## Specification

#### Overview

### Bridge Pools

**Bridge pools** are contracts that hold or hosts tokens being bridged. There are two groups of bridge pools, Native and External and each group has pool logic for each token type.

**Native pools** are pools that bridge tokens from the tokens native chain into alice and they only receive tokens on deposit and send tokens on withdraw.

**External pools** are pools that bridge tokens bridged into alice and then into another external chain, ie token from polygon bridged to alice and then from alice to ethereum. Since the token contract of the token being bridged into a external pool doesnt exist on the current environment, external pools have to act as wrapper tokens for the external token being bridged. When a user withdraws a external token into ethereum, the user will have to burn the UTXO on alicenet, call the bridge router contract with the proof to withdraw, and on succesful validating the proofs supplied a wrapper token will be minted on the external bridge pool.

![Native and External Pool Illustration](/images/natveExternalPool.png)

### Bridge Router
The Bridge Router is a non-upgradeable contract that tracks deposits, withdraws, and emits deposit events of ERC Tokens going into and out of alicent through the native and external bridge pools. 
#### Depositing ERC Tokens
For depositing ERC tokens only Native ERC token deposits must come from BToken since the deposit fees for External Token is charged on exit from alicenet. Each deposit must emit an event with details that uniquely describe the ERC transfer, and a unique nonce must also be emitted in the deposit event for utxo formation.

#### Withdrawing ERC Tokens
All withdraw requests to the bridge pools will originate from this contract, and the hash of the proofs used for the withdraw will be recorded in this contract to prevent multiple withdraws with the same burn proof.  
#### Data

<!-- Data Models / Schemas Requirements -->

#### Logic

<!--- APIs / Pseudocode / Flowcharts / Conditions / Limitations -->

#### Presentation

<!--- UI / UX / Wireframes / Mockups / Design -->

#### Testing

<!--- Testing Requirements -->

#### Security / Risks

<!--- Security / Risks Considerations -->

## Further Considerations

#### Alternative Solutions

<!-- Describe alternative solutions or implementations if any exist -->

#### Timeline

<!--- Estimated timeline to complete / list any milestones -->

#### Prioritization

<!--- How this fits into the roadmap -->

#### Dependencies

<!--- Dependencies on other specs -->

#### Open Questions

<!--- Open questions that need to be answered -->
