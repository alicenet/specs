---
title: Staking ALCA
author: @vtleonardo, @nelsonhp
discussion: https://github.com/alicenet/alicenet/blob/main/bridge/contracts/PublicStaking.sol
team: Backend
category: Development
status: Review
related: -
created: 2022-12-02
---

## Introduction

### Summary

This specification proposes a mechanism to stake the ALCA tokens to aggregate more value to the token and to incentivize users to buy it.

### Context

Once we release the Alicenet economics system, users will be able to get ALCA by migrating the old token (MadToken) directly into the ALCA smart contract. In addition to this, we also foresee the creation of AMM pools with ALCA in popular markets (UNISWAP V3, Balancer, etc), where users will be able to buy ALCA directly using different currencies.

However, a token without a purpose doesn't worth much. In order to add a role for ALCA token to incentivize users to buy it, this specification proposes a staking contract.

### Goals
<!--- What is goal this will accomplish -->

Add a purpose to the ALCA token to generate value.

### Assumptions
<!-- Conditions and resources that need to be present and accessible for the solution to work as described -->

The ALCA contract is deployed and available.

#### Testing
<!--- Testing Requirements -->

- All public functions are tested.
- The whole integration is tested with multiple users, with different staked values and the expected values are correct.

#### Security / Risks
<!--- Security / Risks Considerations -->

If not implemented correctly, users may lose ALCA staked into the system.

## Specification

### Overview
<!--- Describe the solution in detail -->

AliceNet users will be able to stake their ALCA in a smart contract known as `PublicStaking` and participate in the yield distribution of the ALCB selling. As a result of staking an amount $S$ of ALCA in the contract, the users will be receiving a `Non-Fungible Token` (NFT) that describes the properties of the `staked position`. [**Table 1**](#table1) below shows the structure of the `staked positions` NFT.

**<a name="table1">Table 1: Staked Position structure:</a>**

| field             | definition                                                                      |
| ----------------- | ------------------------------------------------------------------------------- |
| tokenID           | unique identifier starting at 1 that will be used to identify a staked position |
| shares            | the number of ALCA staked    in this position                                   |
| freeAfter         | block number after which the position may be burned                             |
| withdrawFreeAfter | block number after which the position may be collected                          |
| accumulatorEth    | the value of the global ether accumulator this account performed withdrawal at  |
| accumulatorToken  | the value of the global token accumulator this account performed withdrawal at  |

The `shares` field accounts for how much `ALCA` was staked in the contract for that position. The `ALCA` staked will be illiquid until the position is burned and due to the nature of the NFTs, the user will not be able to change the amount staked unless he burns the position and re-stakes it. The fields `freeAfter` and `withdrawFreeAfter` can be used to lock a position from being burned. These fields allow multiple uses of staking positions including selling in external markets or even using a staked position as a `future contract` for `ether` and ALCA price. The local accumulators `accumulatorEth` and `accumulatorToken` will be used to compute the yield of a position as will be explained down below.

Each staked NFT position will be identified via a unique identifier called `tokenID_` that is not stored inside the NFT position. The first position will start with the `tokenID=1` and subsequent minted positions will have incremental `tokenIDs`, e.g 2, 3, 4, etc.

Users that choose to stake their `ALCA` (stakeholders) will be gaining yield in the form of `ether` and `ALCA`. The `PublicStaking` contract will be receiving `ether` that will be coming from the direct sell of the ALCB token in the ALCB contract (**NOT FROM ALCB SOLD IN EXTERNAL AMM POOLS!**) and will also be receiving `ALCA` coming from the eventual `slash` of bad validators and from the minting of `ALCA`. The received assets will be stored in a global accumulator per asset (ALCA or `ether`) that can be modeled by [**equation 1**](#eq1) and will be distributed to the stakeholders.

**<a name="eq1">Equation 1: Accumulator logic:</a>**
```math
A_g = \sum_{i=1}^{\infty} {D_i \over T_i}
```

The global accumulator $A_g$ of an asset (`ALCA` or `ether`) will keep track of the summation of all the deposit of that assets $D_i$ divided by total ALCA shares staked in the contract at the moment of the deposit $T_i$.

By using the accumulators defined by [**equation 1**](#eq1) we can compute the yields/profits ( $Y_p$ ) of a position with [**equation 2**](#eq2) down below:

**<a name="eq2">Equation 2: Yield collection equation:</a>**
```math
Y_p = S_p (A_g - A_p)
```

Where, $S_p$ is the shares of the `staked position`, $A_g$ is the current value of the global accumulator of the desired asset (`ether` or `ALCA`), and $A_p$ is the last value of $A_g$ at the time of the last yield collection for a position (`accumulatorEth` and `accumulatorToken` defined in [**table 1**](#table1)).

**During stake position minting, the value of both accumulators will be set to the latest value of the global accumulators ( $A_g = A_p$ ) which will result in $Y_p = 0$ right after the minting**. Therefore, a position will only have yields if a deposit is made after its minting.

Different from the ALCA shares stored inside the `staked position`, the profits (`ALCA` or `ether`) will be liquid. Therefore, the users will receive the assets in the chosen wallets when collecting the profits.

From the point of view of the `PublicStaking` contract, the yield distribution will be passive. In other words, the users will need to call the `PublicStaking` contract in order to collect their profits to the desired wallet address. Passive distribution of yields diminishes a lot the security risks.

> **Users don't need to collect profits right after a distribution has occurred. The collect can occur at any moment, even after multiple distributions, and due to accumulators logic specified in equation [1](#eq1) and [2](#eq2), the value sent to the user will be same as if it has collect right after each distribution.**

## Logic
<!--- APIs / Pseudocode / Flowcharts / Conditions / Limitations -->

### Staking tokens

Users that have `ALCA` will need to `approve` the address of `PublicStaking` contract to transfer the amount wished to stake in the `ALCA` smart contract. After that, users can call the `mint(uint256 amount_)` and `mintTo(address to_, uint256 amount_, uint256 lockDuration_)` in the `PublicStaking` contract to get a new `stake NFT position` in their behalf or to another address (`mintTo`). In the case of `mintTo` users will be also able to specify a lockDuration to protect positions from being burned before a deadline.

### Unstaking tokens

Users with staked position will be able to burn their staked positions by providing the position's `tokenID` to the `burn(uint256 tokenID_)` and `burnTo(address to_, uint256 tokenID_)` in the `PublicStaking` contract. **The burn methods will return to the user the `ALCA` staked in the shares plus any yield of `ether` and `ALCA` owed to that position (see [equation 2](#eq2) for more details).** The assets will be deposited in the user's address or in another desired address (`burnTo`).

### Yield deposit for distribution [**Only External contracts**]

External contracts like `ALCB` and `ValidatorPool` will be allowed to deposit `ether` and `ALCA` to be distributed to the stakeholders via the `depositToken` and `depositEth` methods.

### Estimating the profits

Users will be able to predict their profits without having to call the collect methods. This will allow them to get the amount of the yield owed to a position without having to spend `ether gas` in a transaction. In the `PublicStaking` contract, users will be able to predict the yield by providing the position's `tokenID` to the `estimateEthCollection(uint256 tokenID_)` to estimate only the `ether` profit, `estimateTokenCollection(uint256 tokenID_)` to estimate only the `ALCA` profit, and `estimateAllProfits(uint256 tokenID_)` to estimate both `ether` and `ALCA` profit.

### Collecting profits

In the `PublicStaking` contract, users will be able to collect their yields by providing the position's `tokenID` to the `collectAllProfits(uint256 tokenID_)` and `collectAllProfitsTo(uint256 tokenID_)` to collect both `ALCA` and `ether` profits to their address or to another address (`collectAllProfitsTo`). The `PublicStaking` also has methods to collect each individual assert type to user's address or to another address.

### Locking Staked Positions

Users will be able to lock their position against burn or collection profits. This will allow multiple uses of staking position including selling in external markets or even using a staked position as `future contract` for `ether` and ALCA price. In the `PublicStaking` contract, users can do the locking via the `lockPosition`, `lockOwnPosition`, and `lockWithdraw` methods.

### Data
<!-- Data Models / Schemas Requirements -->

Consider the following scenario:

- `20_000_000 ALCA` staked in the `PublicStaking` contract between `5 distinct users` with positions with different amounts of shares.
- Distribution/Deposit 1 occurs with `1_000_000 ALCA` and `100 ether` from slash events and ALCB selling.

Using equations [1](#eq1) and [2](#eq2), we expect the following results:

<a name="table2">**Table 2: Expected profit of ether and ALCA after the first distribution**</a>

| Owner | tokenID | shares   | % from total staked | profit ether | profit ALCA |
| ----- | ------- | -------- | ------------------- | ------------ | ----------- |
| user1 | 1       | 10000000 | 50%                 | 50           | 500000      |
| user2 | 2       | 5000000  | 25%                 | 25           | 250000      |
| user3 | 3       | 2500000  | 12.5%               | 12.5         | 125000      |
| user4 | 4       | 1500000  | 7.5%                | 7.5          | 75000       |
| user5 | 5       | 1000000  | 5%                  | 5            | 50000       |

Now, let's assume that `user1` burns the position with `tokenID=1`. Now the scenario is:

- `10_000_000 ALCA` staked in the `PublicStaking` contract between `4 distinct users` with positions with different amounts of shares.
- Distribution/Deposit 2 occurs with `2_000_000 ALCA` and `200 ether` from slash events and ALCB selling.

<a name="table3">**Table 3: Expected profit of ether and ALCA after the second distribution**</a>

| Owner | tokenID | shares  | % from total staked | profit ether | profit ALCA |
| ----- | ------- | ------- | ------------------- | ------------ | ----------- |
| user2 | 2       | 5000000 | 50%                 | 100          | 1000000     |
| user3 | 3       | 2500000 | 25%                 | 50           | 500000      |
| user4 | 4       | 1500000 | 15%                 | 30           | 300000      |
| user5 | 5       | 1000000 | 10%                 | 20           | 200000      |

The final user's profit after the 2 distributions can be seen below:

| User  | Total profit ether | total profit ALCA |
| ----- | ------------------ | ----------------- |
| user1 | 50                 | 500000            |
| user2 | 125                | 1250000           |
| user3 | 62.5               | 625000            |
| user4 | 37.5               | 375000            |
| user5 | 25                 | 250000            |

As we can see, having the `ALCA` staked is very beneficial as users will be receiving part of the `ether` profits in the ALCB selling in the ALCB smart contract and `ALCA` coming from slash events. Also, worth noticing that users that stake more will be receiving more.
