---
title: Staking lockup to incentivize illiquidity
author: @vtleonardo, @z-j-lin, @nelsonhp
discussion: https://github.com/alicenet/alicenet/pull/340
team: Backend
category: Development
status: Review
related: -
created: 2022-11-14
---

## Introduction

### Summary

This specification proposes a series of mechanisms to incentivize the illiquidity of ALCA token at the beginning of the AliceNet lifetime. The main component of this mechanism will be a Lockup smart contract that will allow users to lock up their public staking positions in exchange for gaining extra rewards. The lockup smart contract will be assisted by a router smart contract to provide to the end users and to the frontend an easier way of migrating, staking, and locking positions in just one Ethereum transaction.

### Context

Once we release the Alicenet economics system, users will be able to get ALCA by migrating the old token (MadToken) directly into the ALCA smart contract. In addition to this, we also foresee the creation of AMM pools with ALCA in the popular markets (UNISWAP V3, Balancer, etc), where users will be able to buy ALCA directly using different currencies.

With ALCA tokens in hands, users will be able to stake into the Public Staking contract, and in the near future, they will be able participate in the governance system to decide the future of the protocol.

Since ALCA is the main currency of this project, the success of the ALCA is tightly tided with the success of the protocol. At the early lifetime of the project, the ALCA price will be very susceptible to threats such as actors trying to dump/manipulate the price, massive exits by speculation, etc. Therefore, we need a mechanism to incentivize the users to hold their ALCA to have a great illiquidity at the beginning to increase the odds of the AliceNet success.

### Goals
<!--- What is goal this will accomplish -->

Incentive illiquidity of ALCA at the beginning of the project. In other words, incentivize users to lock their ALCA for a certain amount of time until the price becomes more stable.

### Non Goals
<!--- What is not to be included with this -->

Users don't lock their ALCA and dump their directly into the AMM markets.

### Assumptions
<!-- Conditions and resources that need to be present and accessible for the solution to work as described -->

The ALCA and Public Staking contracts are deployed and available and a generous amount of ALCA is separated to be distributed to users that lock their ALCA.

## Specification

### Overview
<!--- Describe the solution in detail -->

To incentivize users to hold their ALCA tokens, a bonus ALCA reward is proposed to users that choose to lock their ALCA staking position in a lockup contract. In other words, a Lockup smart contract will be created and users will be able to transfer their staked positions to it and by doing so, after a certain period has been passed (locking period), they will receive extra rewards. The received reward will be proportional to the amount of ALCA that the user locked compared with the total amount of ALCA locked in the lockup contract.

The bonus reward will be provided by the AliceNet foundation and should be a generous amount to be able to attract users to lock their positions. The bonus amount will be staked in a public staking position to generate additional rewards coming from the redistribution of yield in the ALCB contract. At the end of the locking period, the position will be burned and only users that stayed until the end will receive the bonus reward.

During the locking period, the user will be able to collect profits, coming from the ALCB selling, for the locked position normally. However, the lockup system will hold a certain **reserved** percentage of the collected profit. The reserved amount will be distributed when the user unlocks his position at the end of locking period.

The system will only allow an address per locked position to facilitate the smart contract implementation. The system will also allow users to unlock their positions (partially or fully) before the locking period has finished. Users will able to decide which will be the amount to unlock earlier. In case the user chooses to unlock the total amount, he will not get back the held percentage of profits and he will not receive any bonus amount. In case of partial exit, i.e, when the exit amount is less than the ALCA amount in the locked position, the owner will be loosing only the profits and the bonus relative to the exiting amount. Therefore, users exiting earlier will be penalized by losing the held percentage of profits and any bonus that they would receive. The corresponding amount of held profit and bonus that were owed to the users exiting earlier will be redistributed to the users that stayed locked until the end of the locking period. This protects the system against massive exists and further incentive the users that hold their ALCA in the lockup system by giving them extra rewards.

### Logic
<!--- APIs / Pseudocode / Flowcharts / Conditions / Limitations -->

The lockup system will be composed of 3 smart contracts:

| contract   | definition                                                                                                                                |
| ---------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| Lockup     | Main contract which users will interact to lock, unlock and collect profits                                                               |
| RewardPool | Auxiliary vault contract to store the percentage amount from the locked positions profits (ether and ALCA)                                |
| BonusPool  | Auxiliary vault contract that will stake and hold the bonus ALCA that will be redistributed to the users at the end of the locking period |

The system will work in 4 phases, defined by the time passed in ethereum blocks:

| State           |
| --------------- |
| PreLock         |
| InLock          |
| PostLock-Unsafe |
| PostLock-Safe   |

**PreLock**

The Prelock phase, also called enrollment period, **it is the only phase where the users will be allowed to lock staked positions**. After locking, users can unlock early and collect profits (certain percentage of the profits will be held in the rewards contract). This phase will start right after the lockup contract is deployed and will finish once we reach the ethereum block number that is equal to the value defined at deployment time in the variable: `_startBlock`.

**InLock**

The Inlock phase, also called locked period, is the actual locking period. During this phase, users can unlock early and collect profits (certain percentage of the profits will be held in the rewards contract). This phase will start once we reach the ethereum block number that is equal to the value defined at deployment time in the variable: `_startBlock` and will finish once we reach the ethereum block number defined in the variable `_endBlock`.

**PostLock-Unsafe**

The AliceNet team needs to call a special type of collect function in the lockup contract called `aggregateProfits`. This function will collect all profits for all locked positions. This function different from the normal collect will not send the profits back to the address calling it, but instead, it will store the amount that would go to users in the lockup and the reserved amount (percentage that should be held) in the reward contract. After `aggregateProfits` has finished collecting profits for all positions locked, the bonus position stored in the bonusPool contract will be burned and the assets will be sent to the rewardPool contract.

**PostLock-Safe**



The full logic was already implemented in here: https://github.com/alicenet/alicenet/pull/340


### Data
<!-- Data Models / Schemas Requirements -->

A series of examples were crafted to show the benefits of locking positions.

**The following examples will analyze 5 distinct users with positions with different amount of ALCA staked (shares).**

**The example scenarios will be considering that:**

- 100 million ALCA is staked in total in the Public Staking Contract.
- 1 million ALCA and 100 ether is distributed from slash events and ALCB selling during the locking period.
- 2 million ALCA is given by the AliceNet foundation to be distributed to users that lock their positions (bonus amount).
- 20% of the profits of locked positions are reserved (held) by the lockup system and only given back to the users after the end of the locking period.

> :warning: **The values above are fictitious to ease the reasoning and may change harshly from the real values!**

Before, we dive into the numbers that the users would receive by locking up their public stake positions, let's analyze what they would receive if they never locked their positions:

Table 1: Profit amount of positions not locked.

| Owner | tokenID | shares   | % from total staked | profit ether | profit ALCA |
| ----- | ------- | -------- | ------------------- | ------------ | ----------- |
| user1 | 10      | 10000000 | 10%                 | 10           | 100000      |
| user2 | 20      | 5000000  | 5%                  | 5            | 50000       |
| user3 | 30      | 2500000  | 3%                  | 2.5          | 25000       |
| user4 | 40      | 1500000  | 2%                  | 1.5          | 15000       |
| user5 | 50      | 1000000  | 1%                  | 1            | 10000       |


As we can see in the profits columns, the user's profits are directly proportional to the percentage of ALCA that he has from the total staked.

Table 2: Extra profit that the bonus ALCA would receive for being staked.
| Owner     | tokenID | shares  | % from total staked | profit ether | profit ALCA |
| --------- | ------- | ------- | ------------------- | ------------ | ----------- |
| bonusPool | 51      | 2000000 | 2%                  | 2            | 20000       |

**The table above shows what the bonus amount would receive as profit from being staked. This extra profit will be also distributed to users that lock their positions**

**Example 1: All 5 users lock and stay until the end**

If all the 5 users lock their positions and stay until the end to unlock, we should have in the system:

- 20 million ALCA locked at the end of the locking period.
- 6 ether accumulated from the collects and bonusPosition profit ether.
- 2.06 million ALCA accumulated from the collect + bonus ALCA + bonus position ALCA profit.

Table 2: All 5 users lock their positions and stay until the end.

| owner | tokenID | final locked shares | % from total locked shares | profit ether | profit ALCA |
| ----- | ------- | ------------------- | -------------------------- | ------------ | ----------- |
| user1 | 10      | 10000000            | 50.0%                      | 11           | 1110000     |
| user2 | 20      | 5000000             | 25.0%                      | 5.5          | 555000      |
| user3 | 30      | 2500000             | 12.5%                      | 2.75         | 277500      |
| user4 | 40      | 1500000             | 7.5%                       | 1.65         | 166500      |
| user5 | 50      | 1000000             | 5.0%                       | 1.1          | 111000      |

The final profit in ether can be computed with:

$$ profitEther = {userLockedPositionShares * (accumulatedReservedEther + bonusEther) \over totalSharesLocked} $$

Where:

$userPositionLockedShares$ is the shares (ALCA) of a locked public staking position.
$accumulatedReservedEther$ is the accumulated sum of ether that is held by the lockup system from the profit collection.


In the above example, if all users stayed until the end, user1 would be receiving 1 ether and 1.01 million ALCA extra if he hadn't locked.

#### Presentation
<!--- UI / UX / Wireframes / Mockups / Design -->

#### Testing
<!--- Testing Requirements -->

- All public functions are tested.
- The whole integration is tested with multiple users, with different locked values and the expected values are correct.

#### Security / Risks
<!--- Security / Risks Considerations -->

If not implemented correctly, users may lose ALCA locked up into the system.

## Further Considerations

#### Alternative Solutions
<!-- Describe alternative solutions or implementations if any exist -->

Other alternative solutions were considered, like implementing the reward/lock directly on the public staking contract, but none of them were viable.

#### Timeline
<!--- Estimated timeline to complete / list any milestones -->

Mid November 2022.

#### Prioritization
<!--- How this fits into the roadmap -->

High priority and it needs to be done before the full launch of the project.

#### Dependencies
<!--- Dependencies on other specs -->

No dependencies.

#### Open Questions
<!--- Open questions that need to be answered -->

- How much ALCA will be distributed as bonus?
- What will be the percentage locked in the reward contract?
- How long will be the locking period?
