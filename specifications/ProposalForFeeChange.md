---
title: Proposal for bridge service fees
author: z-j-lin, vtleonardo
discussion: <!--- Discussion URL -->
team: Backend
category: <!--- Development, Informational, User Experience -->
status: <!--- Draft, Review, or Final -->
related: <!-- Issue or Spec [URL] -->
created: 2022-11-17
---

## Introduction

#### Summary

This spec describes the current plans of fee collection for AliceNet bridging, and implications associated with fee collections, as well as possible solutions

#### Context

<!-- Why is this being introduced. Give background and rationale -->

#### Goals

create a competitive fee system for our bridge that is gas efficient, and consistent in all bridges moving tokens into AliceNet

#### Non Goals

<!--- What is not to be included with this -->

#### Assumptions

<!-- Conditions and resources that need to be present and accessible for the solution to work as described -->

## Specification

#### Overview

On the current design, there are fees charged **on alicenet** when users transfer tokens back to other blockchains. The fee will depend on if the destination blockchain is the native blockchain where the token was originated or not.

For the case where users are transfering a token that is native from ethereum into alicenet, they can use the Btoken/ALCB contract directly to minimize the gas costs of the transaction. This happens because ALCB is the currency where the fees are charged, and having the transfering entrypoint into the ALCB contract deminish the number of cross-contract calls optimizing the overall transaction's costs. This method will only work on ethereum since the ALCB contract and the rest of economics systems only exists on ethereum. Therefore, the current problem that this specification in trying to solve is, how the fees will be charged on tokens that are native to other blockchain where ALCB does not exists. In other words, how the fee system can be consistent between all chains.

**Solution 1**

Deploy an ALCB bridge wrapper/external pool on every blockchain supported by AliceNet, except for ethereum. With this approach, users will be able to buy ALCB on ethereum, transfer into AliceNet using a bridge pool so they can have the ALCB wrapped inside AliceNet and then transfer the ALCB out of AliceNet and into the chain their tokens are on.

pros:

- we get deposit fees
- people trying to attack us with shit coins will make eth validators and our validators a lot of money.

cons:

- bridge routers on ethereum will have different logic than routers in other chains
- users will either have to initiate native token deposit on polygon from a external erc20 alcb bridge pool
- vulnerable to Btoken price hike, especially if amm pools come into play, it is in our competitors best interest that BToken price is high on their chain

**solution 2**

charge no deposit fee for native erc tokens going into alicenet, charge withdraw and deposit fee on alicenet when exiting alicenet back to the tokens native chain

pros:

- less friction for onboarding new users
- cheaper than our competitors to enter our chain
- fees are all collected on alicenet reducing the number of cross contract calls
- incentivizes users to stay in our bridge since the overhead for exiting is higher than staying

cons:

- our validators miss out on fees for every erc token deposit
- alcb price could be different from time of entrance and time of exit
- malicious actors can batch transfer a bunch of worthless nfts into alicenet (but the point of layer2 solution is low gas fees so you can mint more useless nfts)

**solution 3**

charge no fees on entrance, the user pays for the gas to transfer token into the bridge pool, and they are charged only a withdraw fee on AliceNet for exiting Alicenet back to the tokens native chain.

pros:

- wayyy more competitive than anyone
- can capture more tokens on our chain, even from polygon
- more users will use out bridge
- we gain more profit from more transactions on alice

cons:

- we miss out on deposit fee

#### Security / Risks

One of the security risk for collecting Btoken on other layer 2 chains is price manipulation of our gas fees to discourage users from leaving their ecosystem.

## Further Considerations

Not collecting ALCB fees for native token deposits means we can remove the deposit token on bridges function in the ALCB contract, and we can deploy ALCB without finalizing the Bridge Router

#### Alternative Solutions

#### Timeline

<!--- Estimated timeline to complete / list any milestones -->

#### Prioritization

<!--- How this fits into the roadmap -->

#### Dependencies

<!--- Dependencies on other specs -->

#### Open Questions

<!--- Open questions that need to be answered -->
