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

# Dutch Auction

## Introduction

### Summary
Stakers looking to become validators should be able to bid
for a position within a
[Dutch auction](https://en.wikipedia.org/wiki/Dutch_auction)
(an open-outcry descending price auction).
The price decreases based on the current number of validators
and the number of validators to be added.

### Context

AliceNet validators prove consensus by signing blocks under a distributed key.
The operation through which this distributed key is generated is called
the ETHDKG (Ethereum Distributed Key Generation) protocol or ceremony;
resulting (public) key is the **group public key**
(the group secret key is never formed).

Because this key is distributed between all validators,
any change in validators will require the negotiation of a new group public key.
Thus, if a validator chooses to leave
and it is desirable for the total validator count to remain the same,
a Dutch auction should be performed to fill the empty position
**before** an ETHDKG ceremony occurs.
Otherwise, two ETHDKG ceremonies would be required:

 *  One to negotiate a key after the validator exits
    (current validators except the one who left)
 *  One to generate a another key after another validator joins
    (current validators except the one who left plus the newly added)

The significant cost of running the ETHDKG protocol
(estimated at approximately 2.5M gas per validator)
necessitates care when executing it;
this protocol should only be performed when desired.
Thus, care will be taken whenever there is a change in the validator set
to ensure that the protocol is performed only when necessary,
as validators bear the cost of running ETHDKG.
In this way, there should be a cost associated when adding validators;
this is done by requiring all potential validators to bid for a position
via a [Dutch auction](https://en.wikipedia.org/wiki/Dutch_auction).

The decision to use a Dutch auction over the more familiar
[English auction](https://en.wikipedia.org/wiki/English_auction)
(an open-outcry ascending price auction)
comes from the advantage that the Dutch auction is over
after the first bid;
thus, it is not necessary to store the bids of potential validators.

### Goals
 *  Establish a mechanism to determine the price to become a validator.

### Non-Goals
 *  The mechanism to become a validator.
 *  The specifics of the ETHDKG protocol.

### Assumptions
Values defined at deployment are a guide
and may be modified in the future.

## Specification

### Overview
The solution consists of a Smart Contract that provides the following functionality:

#### Start an auction
This operation starts an auction by calculating the initial and final prices
and registering the current block number as the auction's start block;
upon execution, an event with all auction information will be emitted.
The execution of this operation will be triggered
only by the sidechain (onlyFactory)
whenever a new validator position is opened. 

#### Get current bidding price
This operation calculates the current bidding price
as determined by the price curve and the number of blocks
since the start of the auction.
The execution of this operation may be performed at any time
to check the current bid price.

#### Bid for current auction price
This operation emits an event with address of the winner
and auction details.
The execution of this operation can be triggered at any time
by any address that pays the current bid price to become a validator. 

### Data
These variables govern the price curve,
and most of them are defined before contract deployment;
only the auction's final price is defined at execution time
since depends on the gas price of the network
and the number of current validators at auction start. 

#### Deployment variables
The following values are set at deployment to produce the results detailed
in the [Testing](#testing) section;
these values may be modified upon contract redeployment to adapt the price curve
to specific requirements.

| Descriptor | Description | Initial Constructor Value |
| ----------- | ----------- |----------- |
| Start Price  | The initial price for auction in Wei | 1000000 * 10 ** 18 |
| Decay | The decay factor (how fast bidding price decreases with time) | 16 |
| Scale Parameter | The scale factor (how curve is compressed) | 100 |

#### Execution variables
The following values are calculated at execution time.

| Descriptor | Description | Calculated Value |
| ----------- | ----------- |----------- |
| Final Price  | The final price for auction in Wei (bidding price can never be less than this value) | Cost to add a validator * Current Number of Registered Validators|

The cost to add a validator is calculated as follows:
 *  ETHDKG Single Validator Cost (Two ETHDKG units operations 1.2M gas units)
    -> 1200000 * 2 operations (In and Out) * gasPriceInWei

### Logic

#### Start Auction
This function will be called when a new validator position has opened;
this may happen when a current validator leaves the network and is replaced
or when the total number of validators of the network is increased.
The following actions are performed:

 *  Define auction final price by multiplying ETHDKG Validator Cost
    by the number of current validators in network
 *  Determine the current auction id by increasing a counter.
 *  Define the auction's start block as the current block number.
 *  Define initial and final price
 *  Upon execution, emit an AuctionStarted event with auction id,
    initial price, and final price

This operation can only be executed by factory.

#### Get Bid Price
This function will be called when a bidder wants to know the current bid price.
The following actions are performed:

 *  Calculate the number of blocks between current block and auction's start block
    (time elapsed since start).
 *  Determine current auction price using the following function:

```solidity
function _dutchAuctionPrice(uint256 blocks) internal view returns (uint256 result) {
    uint256 _alfa = _startPrice - _finalPrice;
    uint256 t1 = _alfa * _scaleParameter;
    uint256 t2 = _decay * blocks + _scaleParameter ** 2;
    uint256 ratio = t1 / t2;
    return _finalPrice + ratio;
}
```

This operation is public.

#### Bid for Current Price
This function will be called when a bidder wants to bid the current price
and earn the validator position.
The following actions are performed:

 *  Bidder includes appropriate ETH amount within transaction.
 *  Emit an event with address of the winner (sender) and auction details.

This operation is public.

### Testing
This is the expected initial bidding price (in ETH):
"1000000.0000000000000000000"

These are the expected price values for the first 5 blocks of the started auction (in ETH):
| Block | Expected Price |
| ----------- | ----------- |
|1|10000.009504000009504000|
|2|9984.035063258795446645|
|3|9968.111577671460859968|
|4|9952.238803821665555414|
|5|9936.416499841026992686|

These are the expected price values for the first 30 days of the started auction (in ETH):
| Day | Expected Price |
| ----------- | ----------- |
|1|10000.009504000009504000|
|2|978.866285982781712764|
|3|514.624662988893909592|
|4|349.074103769906273554|
|5|264.112703317144606972|
|6|212.414015972821832455|
|7|177.642112150073545999|
|8|152.653388985233679619|
|9|133.828247682270598608|
|10|119.136635928723979872|
|11|107.351805925299422928|
|12|97.688742611559180995|
|13|89.621757717408695849|
|14|82.785574306346927859|
|15|76.918477622602351368|
|16|71.828042286708823671|
|17|67.369625219603283677|
|18|63.432401156841503597|
|19|59.930025099477506044|
|20|56.794226720583805605|
|21|53.970316080303145780|
|22|51.413966821574759682|
|23|49.088872370342160681|
|24|46.965010690817608264|
|25|45.017340899444302336|
|26|43.224811339681163975|
|27|41.569595611274020919|
|28|40.036497691258118620|
|29|38.612484036944839430|
|30|37.286312134325050093|

The contract should pass the following tests:

 - [x] Should obtain correct bid price at first auction block
 - [x] Should obtain correct bid prices through five blocks
       according to dutch auction curve
 - [x] Should restart the auction
 - [x] Should get correct prices for the first 30 days

### Presentation

N/A

### Security / Risks

Since in this Contract public users do not modify state, no critical risks are detected

## Further Considerations

### Timeline

Should be released on Q1/2023

### Prioritization

Low / Medium priority.

### Alternative Solutions

N/A

### Dependencies

None

### Open Questions

N/A