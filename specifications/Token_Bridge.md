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

#### Context

#### Goals

Build a low cost multi chain token bridge.

#### Non Goals

<!--- What is not to be included with this -->

#### Assumptions

## Specification

#### Overview

A set of smart contracts deployed on evm compatible chains to transfer tokens between chains. Tokens moved through our bridge will be held and in a bridge pool contract. We will have two different types of pools, a native token and external bridge pool. Native bridge pools are pools that live on the same chain as the token being bridged. When a user deposits a native token into the bridge, they will get and IOU which they can use to withdraw the token on another chain through a external bridge pool. External bridge pools are pools that is deployed on a chain different from the tokens origination chains, and instead of holding tokens, we mint wrapper tokens on exiting chains if a valid IOU is supplied.

![Native and External Pool Illustration](/images/natveExternalPool.png)

#### Transfer Operation Flow

There are currently 2 possible path we could take. The first, the user approving the bridge router to spend the transfer amount, and calls the router contract to initiate the deposit with the required inputs. The router will first burn ALCB fees the user need to pay for the transfer, calls the ERC20 contract to transfer tokens to the bridge pool contracts, then calls the the event emmisions contract to emit an event that notifies the validators to generat an IOU in which the user can use to withdraw the token on another evm chain as a wrapped token. To initiate a withdraw, the user will call the external bride pool contract and burn the token, the bridge pool will call the event emissions contract to notifie the validators to create an IOU for the user to withdraw the token at a specific bridge pool.

the Second path is, similar to the first but instead of having a distinct pool for each token, all tokens are held in a vault contract. The user will approve the bridge router to spend tokens to deposit, them call the bridge router to initiate the deposit with eth required. The bridge router will transfer tokens into the vault contract, calls alcb with eth to do a virtual mint deposit to the 0 address on alice, and returns a IOU for the user to withdraw the token on a external bridge pool contract.

##### Event Emmisions Contract

The events emmissions contract maintains a nonce such that each event emitted will have a unique nonce. To maintain forward compatibility with pool upgrades in the future, this contract must be able to emit events of any arbitrary even encoded data.

##### Preventing double spend

#### Fail Safe

For mitigaiting risks of vulnerabilities, circuit breaker will be implemented that will pause all withdraws and deposits for a period of time. A layered permission system will be used to govern setting and resseting of the circuit breakers(cb). We will have 2 upgradeable proxy contracts that will have the cb setter role and another with the cb resetter role. The cb setter takes calls from the factory and sets the cb on either the bridge or router. The cb resseter contains a function that allows the factory to reset a cb and allows the public to reset the cb after a certain amount of time has elapsed. When a cb is set the current block must be sent to the cb resseter so that it can be used to govern the locking period. The locking period should be long enough for us to solve the problem, but it will prevent us from keeping pools locked indefinitly keeping user fund locked up.

#### Data

<!-- Data Models / Schemas Requirements -->

#### Logic

<!--- APIs / Pseudocode / Flowcharts / Conditions / Limitations -->

#### Presentation

<!--- UI / UX / Wireframes / Mockups / Design -->

#### Testing

<!--- Testing Requirements -->

#### Security / Risks

- Double spending: the hash of the withdraw and deposit data must be tracked if
- concentrated vulnerability: if we decide to store all native toens in a vault, a vulnerability in the vault would risk more tokens simultaneously

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
