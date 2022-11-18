---
title: Proposal for bridge service fees
author: z-j-lin, vtleonardo
discussion: <!--- Discussion URL -->
team: Backend
category: Development
status: Draft
related: <!-- Issue or Spec [URL] -->
created: 2022-11-17
---

## Introduction

AliceNet Bridge allows users to transfer tokens on EVM compatible chains into AliceNet where fees are less expensive.

#### Summary

This spec describes the current plans of fee collection for AliceNet bridging and implications associated with fee collections, as well as possible solutions

#### Context

ALCB is the utility token for AliceNet, all transactions on our layer 2 chain are charged in ALCB, and ALCB is native to Ethereum. There are currently two categories of fees and each category has sub-categories based on what pool logic the user is interacting with. For transferring tokens back into their native chain, we charge a withdrawal fee on AliceNet, and for transferring external tokens into external pools, we charge a withdrawal fee and a deposit fee

#### Goals

create a competitive fee system for our bridge that is gas efficient, and consistent in all bridges moving tokens into AliceNet

#### Non-Goals

#### Assumptions

All Fees are charged in ALCB

## Specification

#### Overview

On the current design, there are fees charged on AliceNet when users transfer tokens back to other blockchains. The fee will depend on if the destination blockchain is the native blockchain of the token.

For the case where users are transferring a token that is native from Ethereum into alicenet, they can use the BToken/ALCB contract directly to minimize the gas costs of the transaction. This is because ALCB is our utility token and it exists on Ethereum. This method will only work on Ethereum since the ALCB contract and the other economic systems only exist on Ethereum. Therefore, the current problem we need to resolve is, having to bridge ALCB to external chains for their native tokens to be transferred to AliceNet

**Solution 1**

Deploy an ALCB bridge wrapper/external pool on every blockchain supported by AliceNet, except for Ethereum. With this approach, users can buy ALCB on Ethereum, transfer it to AliceNet, then bridge it out of AliceNet, into the external chain, and then burn it as a deposit fee to bridge a native token from an external chain into AliceNet.

pros:

- we get deposit fees
- disincentivizes users from flooding AliceNet with useless coins.
- possibly increase fees on other layer 2 chains since gas is cheaper there (for another discussion)

cons:

- bridge routers on Ethereum will have different logic than routers in other chains
- users will either have to initiate native token deposits in external chains from an external ERC20 ALCB bridge pool or interact with a router that calls the external bridge pool.
- vulnerable to Btoken price hike, especially if amm pools come into play, it is in our competitors' best interest that BToken price is high on their chain

**Solution 2**

charge no deposit fee for native erc tokens going into AliceNet, charge withdraw and deposit fee on alicenet when exiting AliceNet back to the tokens native chain

pros:

- less friction for onboarding new users
- cheaper than our competitors to enter our chain
- fees are all collected on AliceNet reducing the number of cross-contract calls
- incentivizes users to stay in our bridge since the overhead for exiting is higher than staying

cons:

- our validators miss out on fees for every ERC token deposit
- ALCB price could be different from the time of entrance and time of exit
- malicious actors can batch transfer a bunch of worthless NFTs into AliceNet (but the point of layer2 solution is low gas fees so you can mint more useless NFTs)

**Solution 3**

charge no fees on entrance, the user pays for the gas to transfer their token into the bridge pool, and they are charged only a withdrawal fee on AliceNet for exiting Alicenet back to the tokens native chain.

pros:

- way more competitive than anyone
- can capture more tokens on our chain, even from polygon
- more users will use our bridge
- we gain more profit from more transactions on AliceNet

cons:

- we miss out on the deposit fee

#### Security / Risks

One of the security risk for collecting Btoken on other layer 2 chains is price manipulation of our gas fees to discourage users from leaving their ecosystem.

## Further Considerations

Not collecting ALCB fees for native token deposits means we can remove the deposit token on bridges function in the ALCB contract, and we can deploy ALCB without finalizing the Bridge Router

#### Alternative Solutions

#### Timeline

<!--- Estimated timeline to complete / list any milestones -->

#### Prioritization

This blocks development of the bridge since this feature touches on nonupgradeable parts of our infrastructure such as the ALCB constract, and Bridge Router.

#### Dependencies

<!--- Dependencies on other specs -->

#### Open Questions

how often can we change our fees?
how are fees calculated?
