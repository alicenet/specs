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
Some constants are calculated using a fix gas price of 100 gweis. In the future this solution should use assembly gasPrice() to get the real gas price of the environment.

## Specification

#### Overview
 <!--- Describe the solution in detail -->
A Smart Contract (DutchAuction.sol) will allow the following operations:
* Start an auction as a privileged user.
* Get auction current bid price as any user
The current bid price will be calculated based of a parameterized curve. 

#### Data
<!-- Data Models / Schemas Requirements -->
These are the constants that will govern the price curve:
* Start Price (-> 1000000 ETH (1000000 * 10 ** 18 weis)
* ETHDKG Single Validator Cost (Two ETHDKG units operations 1.2M gas units each for an estimated 100 gwei gas price) -> 1200000 * 2 * 100 * 10 ** 9 weis
* Decay (Rate of change) -> 16
* Scale Parameter -> 100

#### Logic
<!--- APIs / Pseudocode / Flowcharts / Conditions / Limitations -->

###### Reset Auction
This function will be called when a position for a new validator is open, and will perform the following actions:
* Define auction final price multiplying ETHDKG Validator Cost by the number of current validators in network (The bidding price will never be lower than this value)
* Define auction start block with current block number

###### Get Bid Price
This function will be called when a bidder wants to know the current bid price, the price will be calculated as following: 
* Calculate the number of blocks between current block and auction's start block (time elapsed since start).
* With this number of blocks the price will be determined using the following function:
```solidity
    function _dutchAuctionPrice(uint256 blocks) internal view returns (uint256 result) {
        uint256 _alfa = _START_PRICE - _finalPrice;
        uint256 t1 = _alfa * _SCALE_PARAMETER;
        uint256 t2 = _DECAY * blocks + _SCALE_PARAMETER ** 2;
        uint256 ratio = t1 / t2;
        return _finalPrice + ratio;
    }
```


#### Testing
These are the expected price values for the first 5 blocks of the started auction:
  "10000950400000000000000"
  "9984975974440894568690"
  "9969052503987240829346"
  "9953179745222929936305"
  "9937357456279809220985"

These are the expected price values for the first 30 days of the started auction:
  "10000950400000000000000",
  "979815755677368833202",
  "515574573898723754631",
  "350024172018989109187",
  "265062852313543207268",
  "213364214103653355989",
  "178592343328122779593",
  "153603643912565636829",
  "134778520501017021732",
  "120086922710378347469",
  "108302103907256333190",
  "98639049777291552707",
  "90572072550003584486",
  "83735895636050592675",
  "77868804528394757890",
  "72778374030451019821",
  "68319961200625101040",
  "64382740879801106093",
  "60880368151095345381",
  "57744572752464452823",
  "54920664796028491258",
  "52364317966854463955",
  "50039225725391652597",
  "47915366064385259757",
  "45967698124077341302",
  "44175170267934312878",
  "42519956112644213186",
  "40986859649684588043",
  "39562847348753898891",
  "38236676706527897891",

The solution should pass the following tests:
✓ Should obtain correct bid price at first auction block
✓ Should obtain correct bid prices through five blocks according to dutch auction curve
✓ Should restart the auction
✓ Should get correct prices for the first 30 days

#### Presentation

N/A

#### Security / Risks

Since this Contract does not modify state  

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