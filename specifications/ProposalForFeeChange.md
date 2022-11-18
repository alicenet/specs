---
title: Proposal for bridge service fees
author: @z-j-lin
discussion: <!--- Discussion URL -->
team: Backend
category: <!--- Development, Informational, User Experience -->
status: <!--- Draft, Review, or Final -->
related: <!-- Issue or Spec [URL] -->
created: 2022-11-17
---

## Introduction

#### Summary

This spec is a proposal to modify change types of fees we collect for our bridging services

#### Context

<!-- Why is this being introduced. Give background and rationale -->

#### Goals

create a competitive fee system for our bridge that is gas efficient

#### Non Goals

<!--- What is not to be included with this -->

#### Assumptions

<!-- Conditions and resources that need to be present and accessible for the solution to work as described -->

## Specification

#### Overview

we charge users a withdraw fee **on alicenet** for transfering tokens from alicenet back to their native chains, and we charge users a withdraw, and reentrance fee **on alicenet** for external tokens entering a external chain from Alicenet.
for the case where we are transfering a token native to ethereum into alicenet we only allow users to transfer through the btoken contract so that the depositFee can be paid directly to the gas token contract iff the deposit is successful. This method will only work on ethereum since the ALCB contract only exists on ethereum.

We currently have 3 solutions for this problem

solution 1: deploy a ALCB bridge wrapper/external pool so that users can transfer ALCB out of alice and into the chain their tokens are on.

pros:

- we get deposit fees
- people trying to attack us with shit coins will make eth validators and our validators a lot of money.

cons:

- bridge routers on ethereum will have different logic than routers in other chains
- users will either have to initiate native token deposit on polygon from a external erc20 alcb bridge pool

solution 2: charge no deposit fee for native erc tokens going into alicenet, charge all the fees on exit in alicenet

pros:

- less friction for onboarding new users
- cheaper than our competitors to enter
- incentivizes users to stay in our bridge

cons:

- our validators miss out on fees for every erc token deposit
- malicious actors can batch transfer a bunch of worthless nfts into alicenet (but the point of layer2 solution is low gas fees so you can mint more useless nfts)

solution 3: charge no fees on entrance, the user pays for the gas to transfer token into the bridge pool, and they get charged only a withdraw fee for going from alice back into native chain.

pros:

- wayyy more competitive than polygon
- can capture more tokens on our chain, even from polygon
- more users will use out bridge
- we gain more profit from more transactions on alice

cons:

- we miss out on deposit fee

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
