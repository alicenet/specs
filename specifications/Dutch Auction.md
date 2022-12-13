---
title: Dutch Auction Smart Contract
author: Gustavo Vazquez
discussion: 
team: Backend
category: Development
status: Draft
related: https://github.com/alicenet/alicenet/issues/398
created: 2022-12-07
---

## Introduction

#### Summary   
Stakers looking to become a validator should be able to bid for joining price in an dutch style auction.
The price curve should take into account the need for new validator in both the starting price and the rate of change.

#### Context
AliceNet validator nodes communicate with each other in a secure way through the use of a distributed key.
The operation through which this distributed key is generated is called ETHDKG (Ethereum Distribute Key Generation) ceremony.

When a new validator is admitted to join AliceNet, two ETHDKG ceremonies must be executed:
* One to destroy the current distributed key (all current validators)
* One to generate a new distributed key (all current validators plus the new one)

Since this operation involves interacting with multiple Ethereum Smart Contract functions it is a costly operation estimated in approx 2.5M gas units for each validator.
To encourage current validators incur in this expense whenever a new validator wants to join the network, an amount to be paid by the new validator will be required.
This amount will be determined by an auction among the postulants, the one that wins pays the price and is allowed to join the network, the minimum price for this auction must cover at least the expenses of all current validators.

Regular auction systems need to maintain the auction open for a fixed period of time, adding complexity and creating sometimes unnecessary waits. There is a simpler and faster auction system called Dutch Auction where the bidding initial price decreases automatically with time following a parameterized curve until a bid is placed, at this point the bidder wins and the auction is closed.

#### Goals
Establish a mechanism to determine the price to become a validator.

#### Non Goals
The mechanism to become a validator.

#### Assumptions
Values defined at deployment are defined as a guide and can be modified to adapt the price curve to required results.

## Specification

#### Overview
The solution consists of a Smart Contract that provides the following functionality:
##### Start an auction
This operation allows to start an auction by calculating the initial and final prices and registering the current block number as auction's start block, upon execution, an event with all auction details will be emitted.
##### Get current bidding price
This operation calculates the current bidding price using a specific price curve and the number of blocks mined between current block and the registered start block.
##### Bid for current auction price
This operation emits an event with address of the winner (sender) and auction details

#### Data
These variables will govern the price curve, the values are set at deployment
| Descriptor | Constructor Value |
| ----------- | ----------- |
| Start Price  | 1000000 * 10 ** 18 |
| Decay | 16 |
| Scale Parameter | 100 |

The cost to add a validator is calculated as follows:
* ETHDKG Single Validator Cost (Two ETHDKG units operations 1.2M gas units) -> 1200000 * 2 * gasPriceInWeis

Auction's final price can be obtained multiplying the current number of registered validators by ETHDKG Single Validator Cost.

#### Logic
##### Start Auction
This function will be called when a position for a new validator is open, and performs the following actions:
* Defines auction final price multiplying ETHDKG Validator Cost by the number of current validators in network (The bidding price will never be lower than this value)
* Determines the current auction id by increasing a counter.
* Defines the auction's start block as the current block number.
* Defines initial and final price
* Upon execution emit an AuctionStarted event with auction id, initial price and final price
This operation can only be executed by factory

##### Get Bid Price
This function will be called when a bidder wants to know the current bid price, and performs the following actions: 
* Calculates the number of blocks between current block and auction's start block (time elapsed since start).
* Determines current auction price using the following function:
```solidity
    function _dutchAuctionPrice(uint256 blocks) internal view returns (uint256 result) {
        uint256 _alfa = _startPrice - _finalPrice;
        uint256 t1 = _alfa * _scaleParameter;
        uint256 t2 = _decay * blocks + _scaleParameter ** 2;
        uint256 ratio = t1 / t2;
        return _finalPrice + ratio;
    }
```
This operation is public

##### Bid for Current Price
This function will be called when a bidder wants to buy for the bidding price, and performs the following actions: 
* Emits an event with address of the winner (sender) and auction details
This operation is public

#### Testing
This is the expected initial bidding price (in ETH):
"1000000.0000000000000000000"

These are the expected price values for the first 5 blocks of the started auction (in ETH):
1. "10000.009504000009504000"
2. "9984.035063258795446645"
3. "9968.111577671460859968"
4. "9952.238803821665555414"
5. "9936.416499841026992686"

These are the expected price values for the first 30 days of the started auction (in ETH):
1. "10000.009504000009504000"
2. "978.866285982781712764"
3. "514.624662988893909592"
4. "349.074103769906273554"
5. "264.112703317144606972"
6. "212.414015972821832455"
7. "177.642112150073545999"
8. "152.653388985233679619"
9. "133.828247682270598608"
10. "119.136635928723979872"
11. "107.351805925299422928"
12. "97.688742611559180995"
13. "89.621757717408695849"
14. "82.785574306346927859"
15. "76.918477622602351368"
16. "71.828042286708823671"
17. "67.369625219603283677"
18. "63.432401156841503597"
19. "59.930025099477506044"
20. "56.794226720583805605"
21. "53.970316080303145780"
22. "51.413966821574759682"
23. "49.088872370342160681"
24. "46.965010690817608264"
25. "45.017340899444302336"
26. "43.224811339681163975"
27. "41.569595611274020919"
28. "40.036497691258118620"
29. "38.612484036944839430"
30. "37.286312134325050093"

The contract should pass the following tests:
✓ Should obtain correct bid price at first auction block
✓ Should obtain correct bid prices through five blocks according to dutch auction curve
✓ Should restart the auction
✓ Should get correct prices for the first 30 days

#### Presentation

N/A

#### Security / Risks

Since in this Contract public users do not modify state, no critical risks are detected

## Further Considerations

#### Timeline

Should be released on Q1/2023

#### Prioritization

Low / Medium priority.

#### Alternative Solutions

N/A

#### Dependencies

None

#### Open Questions

N/A