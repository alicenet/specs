---
title: Staking lockup to incentivize illiquidity
author: @vtleonardo, @nelsonhp
discussion: https://github.com/alicenet/alicenet/pull/340
team: Backend
category: Development
status: Review
related: -
created: 2022-11-14
---

## Introduction

### Summary

This specification proposes a series of mechanisms to incentivize the illiquidity of ALCA token at the beginning of the AliceNet lifetime. The main component of this mechanism will be a Lockup smart contract that will allow users to lock their public staking positions in exchange for gaining extra rewards.

### Context

Once we release the Alicenet economics system, users will be able to get ALCA by migrating the old token (MadToken) directly into the ALCA smart contract. In addition to this, we also foresee the creation of AMM pools with ALCA in popular markets (UNISWAP V3, Balancer, etc), where users will be able to buy ALCA directly using different currencies.

With ALCA tokens in hand, users will be able to stake into the Public Staking contract, and in the near future, they will be able to participate in the governance system to decide the future of the protocol.

Since ALCA is the main currency of this project, the success of the ALCA is tightly tied to the success of the protocol. In the early life of the project, the ALCA price will be very susceptible to threats such as actors trying to dump/manipulate the price, massive exits by speculation, etc. Therefore, we need a mechanism to incentivize the users to hold their ALCA to have great illiquidity at the beginning to increase the odds of AliceNet success.

### Goals
<!--- What is goal this will accomplish -->

Incentive illiquidity of ALCA at the beginning of the project. In other words, incentivize users to lock their ALCA for a certain amount of time until the price becomes more stable.

### Non Goals
<!--- What is not to be included with this -->

Users don't lock their ALCA and dump them directly into the AMM markets.

### Assumptions
<!-- Conditions and resources that need to be present and accessible for the solution to work as described -->

The ALCA and Public Staking contracts are deployed and available and a generous amount of ALCA is separated to be distributed to users that lock their ALCA.

## Overview
<!--- Describe the solution in detail -->

To incentivize users to hold their ALCA tokens, a bonus ALCA reward is proposed to users that choose to lock their ALCA staking position in a Lockup contract for a certain `locking period`. The Lockup contract will work as a custodial wallet during the `locking period`. Users will be able to collect profits and exit at any moment. However, only users that stay until the end of the `locking period` will be receiving rewards. The received reward will be proportional to the amount of ALCA that the user locked compared with the total amount of ALCA locked in the lockup contract.

The bonus reward will be provided by the AliceNet foundation and should be a generous amount to be able to attract users to lock their positions. Right now, the bonus amount will be **900000+ ALCA**. The bonus amount $S(b)$ will be staked in a public staking position to generate additional rewards coming from the redistribution of yield in the ALCB contract and from the minting of ALCA. At the end of the locking period, the position will be burned and only users that stayed until the end will receive the bonus reward. We can define the total bonus amount of `ALCA` ( $B_{alca}$ ) and `ether` ( $B_{eth}$ ) to be distributed as such:

$$ B_{alca} = S(b) + Y(b)_{alca} \tag{1}$$

and

$$ B_{eth} = Y(b)_{eth} \tag{2}$$

Where $S_b$ is the total amount of `ALCA` staked by the AliceNet foundation and $Y_{eth}$ and $Y_{alca}$ are the profits in `ether` and `ALCA` that occurred during the locking period.

During the locking period, the user will be able to collect profits, coming from the ALCB selling and ALCA minting for the locked position normally. However, the lockup system will hold a certain **reserved** percentage of the collected profit. The **reserved percentage will be 20%**. The reserved amount will be stored in an auxiliary contract called `RewardPool` and it will be distributed when the user unlocks his position at the end of the locking period.

Using the staking accumulator and user profit equations defined in the [staking specs](https://github.com/alicenet/specs/pull/31), we can derive that the held profit in `ALCA` ( $H_{alca}$ ) and in `ether` ( $H_{eth}$ ) of a locked position will be:

$$ H(u)_{alca} = 0.2Y(u)_{alca}  \tag{3}$$

and

$$ H(u)_{eth} = 0.2Y(u)_{eth}  \tag{4}$$

Similarly, we can define that liquid profit in `ALCA` ( $L_{alca}$ ) and `ether` ( $L_{eth}$ ) that a user would receive by collecting profits from a locked position during the locking period will be:

$$ L(u)_{alca} = 0.8Y_{alca} \tag{5}$$

and

$$ L(u)_{eth} = 0.8Y_{eth} \tag{6}$$

The [accumulator logic](https://github.com/alicenet/specs/pull/31) of the staking system makes it possible that $H$ and $L$ will be the same if the user collects multiple times during the `locking period` or just once.

The system will only allow an address per locked position to facilitate the smart contract implementation and all the positions will be tracked inside the smart contract in an indexed data struct.

Assuming that we have $N$ positions locked in the Lockup system, the total reserved amount $H$ stored in the `RewardPool` can be defined by the summation of the held profit of all positions locked in the system:

$$ H_{alca} = \sum_{u=1}^{N} H(u)_{alca} \tag{7}$$

and

$$ H_{eth} = \sum_{u=1}^{N} H(u)_{eth} \tag{8}$$

In order to ensure that $(7)$ and $(8)$ hold true and that each user receives the correct held amount $H(u)$ owed to them, we have to make sure that the profit is collected for all locked positions. This will be enforced at the end of the `locking period`. Once the `locking period` is over, a special method called `aggregateProfits()` in the `Lockup` contract needs to be called. This method will iterate over all locked positions collecting the profits. Differently from the normal collection of profits that can happen during the `locking period`, the collection of profits of the `aggregateProfits()` will not send the liquid profit $G(u)$ to users. This value will be stored in the `Lockup` contract and will be sent together with the final bonus and held amount. With this, we can allow anyone to call `aggregateProfits()` after the `locking period` is over. Probably, it will be necessary more than one call to collect the profits for all positions due to the gas limits imposed by the Ethereum blockchain.

At the end of the `locking period` and after `aggregateProfits` has been completed, the final outcome $R(u)$ received by a user when unlocking will be:

$$ R(u)_{alca} = {S(u)\over S_{T}} (H_{alca} + B_{alca}) + S(u) + G(u)_{alca} \tag{9}$$

and

$$ R(u)_{eth} = {S(u)\over S_{T}} (H_{eth} + B_{eth} ) + G(u)_{eth} \tag{10}$$

Where, $S_T$ is the total number of shares (ALCA) locked into the contract at the end of the `locking period` and $G(u)$ is the liquid profit for a user stored in the `Lockup` contract during the `aggregateProfits()` execution.

The system will also allow users to unlock their positions (partially or fully) before the locking period has finished. Users will be able to decide which will be the amount to unlock earlier. In case the user chooses to unlock the total amount, he will not get back any held percentage of profits $H(u)$ previously stored and he will not receive any bonus amount. During the `early unlock`, a collection of profits will be done and the reserved amount $H(u)$ will be deducted from the yield before delivering the `staked position` back to the user.

In case of partial exit, i.e, when the exit amount is less than the ALCA amount in the locked position, the owner will be losing only the profits and the bonus relative to the exiting amount. This will change $S(u)$ and also $S_T$ changing the final outcome received by the users.

Therefore, users exiting earlier will be penalized by losing the held percentage of profits and any bonus that they would receive. The corresponding amount of held profit and bonus that were owed to the users exiting earlier will be redistributed to the users that stayed locked until the end of the locking period following equations $(9)$ and $(10)$. I.e, users exiting early will decrease the value of $S_T$, increasing the profits of the users locked. This protects the system against massive exists and further incentive the users that hold their ALCA in the lockup system by giving them extra rewards.

During the `unlock early` or the final `unlock`, users will be able to decide if they want to receive the `ALCA` amount directly as tokens on their wallets or as a new `staked position` in the `Public Staking` contract.

## Logic
<!--- APIs / Pseudocode / Flowcharts / Conditions / Limitations -->

The full logic was already implemented here: https://github.com/alicenet/alicenet/blob/main/bridge/contracts/Lockup.sol

The lockup system will be composed of 3 smart contracts:

| Contract   | Definition                                                                                                                                |
| ---------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| Lockup     | Main contract which users will interact to lock, unlock and collect profits                                                               |
| RewardPool | Auxiliary vault contract to store the percentage amount from the locked positions profits (ether and ALCA)                                |
| BonusPool  | Auxiliary vault contract that will stake and hold the bonus ALCA that will be redistributed to the users at the end of the locking period |

The system will work in 4 phases, defined by the time passed in ethereum blocks:

| State           | Definition                                                   |
| --------------- | ------------------------------------------------------------ |
| PreLock         | Enrollment period where users will be able to lock positions |
| InLock          | Locking period                                               |
| PostLock-Unsafe | Post-Lock period where we need to run `aggregateProfits()`   |
| PostLock-Safe   | Post-Lock period where users can `unlock` without penalties  |

**PreLock**

The Prelock phase, also called the **enrollment period**, **is the only phase where the users will be allowed to lock staked positions**. After locking, users can unlock early and collect profits (a certain percentage of the profits will be held in the rewards contract). This phase will start right after the lockup contract is deployed and will finish once we reach the ethereum block number that is equal to the value defined at deployment time in the variable: `_startBlock`. The PreLock phase will last approximately 3 months.

**InLock**

The Inlock phase, also called the locked period, is the actual locking period. During this phase, users can unlock early and collect profits (a certain percentage of the profits will be held in the rewards contract). This phase will start once we reach the ethereum block number that is equal to the value defined at deployment time in the variable: `_startBlock` and will finish once we reach the ethereum block number defined in the variable `_endBlock`. The InLock phase will last approximately 12 months.

**PostLock-Unsafe**

During the PostLock-Unsafe phase, users will not be able to collect profits nor unlock earlier. During this phase, the AliceNet team will need to call a special type of collect function in the lockup contract called `aggregateProfits`. This function will collect all profits for all locked positions. This function different from the normal collect will not send the profits back to the address calling it, but instead, it will store the amount that would go to users in the lockup and the reserved amount (the percentage that should be held) in the reward contract. After `aggregateProfits` has finished collecting profits for all positions locked, the bonus position stored in the bonusPool contract will be burned and the assets will be sent to the rewardPool contract.

**PostLock-Safe**

In the PostLock-Safe phase, users will be able to `unlock()` their positions without any penalty and will gain the final reward described by equations (9) and (10). Users will be able to decide if they want the `ALCA` tokens to be sent directly to their desired wallet or to be re-staked in a new `staking position` in the Public Staking contract.

## Testing
<!--- Testing Requirements -->

- All public functions are tested.
- The whole integration is tested with multiple users, with different locked values and the expected values are correct.

## Security / Risks
<!--- Security / Risks Considerations -->

If not implemented correctly, users may lose ALCA locked up into the system.

## Further Considerations

### Alternative Solutions
<!-- Describe alternative solutions or implementations if any exist -->

Other alternative solutions were considered, like implementing the reward/lock directly on the public staking contract, but none of them were viable.

### Timeline
<!--- Estimated timeline to complete / list any milestones -->

Mid December 2022.

### Prioritization
<!--- How this fits into the roadmap -->

High priority and it needs to be done before the full launch of the project.

### Dependencies
<!--- Dependencies on other specs -->

Depends on https://github.com/alicenet/specs/pull/31.
