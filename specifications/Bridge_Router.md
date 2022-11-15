---
title: Bridge Router 
author: <!--- Author Name(s) -->
discussion: <!--- Discussion URL -->
team: <!--- Backend or Frontend -->
category: <!--- Development, Informational, User Experience -->
status: <!--- Draft, Review, or Final -->
related: <!-- Issue or Spec [URL] -->
created: <!--- YYYY-MM-DD -->
---

## Introduction

#### Summary   
<!--- Summary of the spec. Describe what it does -->

#### Context
<!-- Why is this being introduced. Give background and rationale -->

#### Goals
<!--- What is goal this will accomplish -->

#### Non Goals
<!--- What is not to be included with this -->

#### Assumptions
- only native ERC token deposits will require BToken Fees to be collected on layer 1
- Bridge pools are deployed at deterministic locations with metamorphic initialization code, as EIP 1167 thin proxies. 
- bridge pool salts are calculated as follows
- all bridge pools have a deposit and withdraw function that only this contract can call
 
``` solidity
  /**
     * @notice calculates salt for a BridgePool contract based on ERC contract's address, tokenType, chainID and version_
     * @param tokenContractAddr_ address of ERC contract of BridgePool
     * @param tokenType_ type of token (0=ERC20, 1=ERC721, 2=ERC1155)
     * @param version_ version of the implementation
     * @param chainID_ chain ID
     * @return calculated calculated salt
     */
    function getBridgePoolSalt(
        address tokenContractAddr_,
        uint8 tokenType_,
        uint256 chainID_,
        uint16 version_
    ) internal pure returns (bytes32) {
        return
            keccak256(
                bytes.concat(
                    keccak256(abi.encodePacked(tokenContractAddr_)),
                    keccak256(abi.encodePacked(tokenType_)),
                    keccak256(abi.encodePacked(chainID_)),
                    keccak256(abi.encodePacked(version_))
                )
            );
    }
```
## Specification

#### Overview
All erc token deposits and withdraws in and out of alicenet must go through the bridge router. For all ERC token deposits into alicenet a deposit event must be emitted with details specific to the bridge pool being used, and a monotonically increasing nonce for each deposit.
To withdraw a from alicenet, the utxo that describes the token must be burned, and the user is given a proof to be used with snapshot data for deposits. 
the hash of the proofs used must be stored in a mapping to prevent it from being used in the future.

since this contract hold privilidges that allows it to move erc tokens in bridge pools, it must not be upgradeable. 

since the router is the central place where users interact with the bridge pools, it must have a circuit breaker that allows a contract with role circuit breaker to set it. 


DepositExternalToken

WithdrawTokens

#### Data



<!-- Data Models / Schemas Requirements -->
For forward compatibility with future pools a dynamic data structure should be used for input data.


```solidity 

struct TransactionData {
      address poolAddress;
      uint16 poolVersion;
      uint8 poolType;
      uint8 ercType;
      bytes txData;
  }
```

#### Logic
``` solidity 
function _emitDepositEvent(bytes32[] memory topics_, bytes memory eventData_) internal {
        if (topics_.length == 1) _log1Event(topics_, eventData_);
        else if (topics_.length == 2) _log2Event(topics_, eventData_);
        else if (topics_.length == 3) _log3Event(topics_, eventData_);
        else if (topics_.length == 4) _log4Event(topics_, eventData_);
        else revert CentralBridgeRouterErrors.MissingEventSignature();
    }

    function _log1Event(bytes32[] memory topics_, bytes memory eventData_) internal {
        bytes32 topic0 = topics_[0];
        assembly {
            log1(add(eventData_, 0x20), mload(eventData_), topic0)
        }
    }

    function _log2Event(bytes32[] memory topics_, bytes memory eventData_) internal {
        bytes32 topic0 = topics_[0];
        bytes32 topic1 = topics_[1];
        assembly {
            log2(add(eventData_, 0x20), mload(eventData_), topic0, topic1)
        }
    }

    function _log3Event(bytes32[] memory topics_, bytes memory eventData_) internal {
        bytes32 topic0 = topics_[0];
        bytes32 topic1 = topics_[1];
        bytes32 topic2 = topics_[2];
        assembly {
            log3(add(eventData_, 0x20), mload(eventData_), topic0, topic1, topic2)
        }
    }

    function _log4Event(bytes32[] memory topics_, bytes memory eventData_) internal {
        bytes32 topic0 = topics_[0];
        bytes32 topic1 = topics_[1];
        bytes32 topic2 = topics_[2];
        bytes32 topic3 = topics_[3];
        assembly {
            log4(add(eventData_, 0x20), mload(eventData_), topic0, topic1, topic2, topic3)
        }
    }
```

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




