---
title: Including Tokens on AliceNet
author: @kosegor
discussion: https://github.com/alicenet/research/blob/master/upgrades/tokens/discussion.pdf
team: Backend
category: Development
status: Draft
related: https://github.com/alicenet/specs/issues/39
created: 2023-01-30
---

## Introduction

#### Summary   
This document proposes a specification for tokens inclusion on the AliceNet as a new UTXO type. 

#### Context
Currently, AliceNet accepts only 2 types of UTXO: ValueStore and DataStore. In order to deposit, transfer and withdraw Tokens a new UTXO type must be introduced. The purpose of enabling Tokens on AliceNet is to facilitate their transfer between parties, making possible for AliceNet users to move their different assets across the chain without any third party intervention.

#### Goals
 - Add a new UTXO type called Token.
 - Token can be deposited within a transaction in a very similar way ValueStore does and assigned to a depositor account.
 - Token can be transferred from one account to another within a transaction in a very similar way ValueStore does.
 - Token can be withdrawn within a transaction. This is something new that is not implemented for any type of UTXO on AliceNet. 

#### Non Goals
 - Swap one Token for another Token within AliceNet chain. This can be done only outside the chain, on AliceNet the conversion from one tokenID to another must not be allowed.

#### Assumptions
<!-- Conditions and resources that need to be present and accessible for the solution to work as described -->

## Specification

### Overview
The solution consists of a new UTXO type called Token. The Token should meet the following conditions in order to be included in the Transaction as a valid TxOut type:

#### Validation
The Token must be able to valid itself based on the [Type Definition](#type-definition).

#### Deposit
Token deposits must be identified in a unique way assigning an utxoID. A deposit could be spent only once, after that it must be stored in the State Trie.

#### Transfer
A Token UTXO could be transferred fully or partially to another owner within a transaction. This is very similar to ValueStore object.

#### Withdraw
A Token UTXO could be withdrawn entirely to the origin chain within a transaction. This cannot be done multiple times, the withdrawal login must take care of that. 

### Data
#### Token Data Model:
 - Token
   - TPreImage
     - ChainID - uint32
     - ExitChainID - uint32
     - SmartContract
       - OriginChainID - uint32
       - Address - []byte
     - TokenID - uint256
     - Value - uint256
     - Withdraw - bool
     - TxOutIdx - uint32
     - Owner
       - SVA - SVA
       - CurveSpec - constants.CurveSpec
       - Account - []byte
     - Fee - uint256

#### Type Definition:

We build our Token object by extending the standard ValueStore object; however, we use a separate UTXO type to make validation easier and to ensure there is no confusion. This is a completely different object and should be treated as such.

The proposed Token type definition requires Value to be in expressed in the amount of token; it must be nonzero. In order to distinguish different tokens, we include a reference to the originating smart contract (including reference to the smart contract address as well as the chain ID of the origination; this will ensure that tokens with the same smart contract address on different blockchains are different tokens) as well as a TokenID parameter. The Withdraw flag ensures that, when set to true, the token must be withdrawn; its value can no longer be transferred. The ExitChainID tracks to which blockchain the tokens will be withdrawn.

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




