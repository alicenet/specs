---
title: Proposal for bridge service fees
author: z-j-lin, vtleonardo
discussion: https://github.com/alicenet/specs/discussions/20
team: Backend
category: Development
status: Draft
related: [Bridge Router](https://github.com/alicenet/alicenet/issues/478) , [Bridge Pool Base](https://github.com/alicenet/specs/pull/16)
created: 2022-11-17
---

## Introduction

AliceNet Bridge allows users to transfer tokens on EVM compatible chains into AliceNet where fees are less expensive.

#### Summary

This specification describes the current plans of fee collection for AliceNet bridging and implications associated with fee collections, as well as possible solutions

#### Context

ALCB is the ERC20 utility token for AliceNet, all transactions on our layer 2 chain are charged in ALCB, and ALCB is native to Ethereum. There are currently two categories of fees and each category has sub-categories based on what pool logic the user is interacting with. For transferring tokens back into their native chain, we charge a withdrawal fee on AliceNet, and for transferring external tokens into external pools, we charge a withdrawal fee and a deposit fee.

When ALCB is deposited on Alicenet it is burned on Ethereum, it can never come back to ethereum, and the Eth from the sale of the ALCB is free to be split by the distribution contract. When ALCB is used on Ethereum for bridge token deposit, it is also burned.

#### Goals

Create a competitive fee system for our bridge that is gas efficient, and consistent in all bridges moving tokens into AliceNet

#### Non-Goals

#### Assumptions

- All Fees are charged in ALCB
- We want people to transfer high trade volume tokens into AliceNet
- Our First road block in attracting current crypto users into our ecosystem is the onboarding overhead. The price of transfering their existing coins could be a point of friction for users considering moving their tokens into AliceNet
- The amount of high trade volume tokens in our system is directly proportional to the profitability of our system
- Most users will determine what bridge they will use based on the transfer cost

## Specification

#### Overview

On the current design, there are fees charged on AliceNet when users transfer tokens back to other blockchains. The fee will depend on if the destination blockchain is the native blockchain of the token.

For the case where users are transferring a token that is native from Ethereum into alicenet, they can use the BToken/ALCB contract directly to minimize the gas costs of the transaction. This is because ALCB is our utility token and it exists on Ethereum. This method will only work on Ethereum since the ALCB contract and the other economic systems only exist on Ethereum. Therefore, the current problem we need to resolve is, having to bridge ALCB to external chains for their native tokens to be transferred to AliceNet

**Solution 1**

Deploy an ALCB bridge wrapper/external pool on every blockchain supported by AliceNet, except for Ethereum. With this approach, users can buy ALCB on Ethereum, transfer it to AliceNet, then bridge it out of AliceNet, into the external chain, and then burn it as a deposit fee to bridge a native token from an external chain into AliceNet.

Pros:

- We get deposit fees
- Disincentivizes users from flooding AliceNet with useless coins.
- Possibly increase fees on other layer 2 chains since gas is cheaper there (for another discussion)

Cons:

- Bridge routers on Ethereum will have different logic than routers in other chains
- Users will either have to initiate native token deposits in external chains from an external ERC20 ALCB bridge pool or interact with a router that calls the external bridge pool.
- Vulnerable to Btoken price hike, especially if amm pools come into play, it is in our competitors' best interest that BToken price is high on their chain

**Solution 2**

Charge no deposit fee for native erc tokens going into AliceNet, charge withdraw and deposit fee on alicenet when exiting AliceNet back to the tokens native chain

Pros:

- Less friction for onboarding new users
- Cheaper than our competitors to enter our chain
- Fees are all collected on AliceNet reducing the number of cross-contract calls
- Incentivizes users to stay in our bridge since the overhead for exiting is higher than staying

Cons:

- Our validators miss out on fees for every ERC token deposit
- ALCB price could be different from the time of entrance and time of exit
- Malicious actors can batch transfer a bunch of worthless NFTs into AliceNet (but the point of layer2 solution is low gas fees so you can mint more useless NFTs)

**Solution 3**

Charge no fees on entrance, the user pays for the gas to transfer their token into the bridge pool, and they are charged only a withdrawal fee on AliceNet for exiting Alicenet back to the tokens native chain.

Pros:

- Way more competitive than anyone
- Can capture more tokens on our chain, even from polygon
- More users will use our bridge
- We gain more profit from more transactions on AliceNet

Cons:

- We miss out on the deposit fee

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

BridgeRouter, and Bridge pool specs depends on the result of the discussion associated with this spec

#### Open Questions

how often can we change our fees?
how are fees calculated, is it per token or flat fee per transfer?
whats the difference between ERC20 fees and ERC721 fees?
How are we calculating fees for ERC1155 batch transfers?
