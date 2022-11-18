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
- lock and unlock logic is abstracted in the bridgeCBSetter and bridgeCBResetter upgradeable proxy contracts
- Bridge pools are deployed at deterministic locations where the salt =

```solidity
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

The Bridge Router is a non-upgradeable contract that tracks deposits, withdraws, and emits deposit events of ERC Tokens going into and out of alicent through the native and external bridge pools.

#### Depositing ERC Tokens

For depositing ERC tokens only Native ERC token deposits must come from BToken since the deposit fees for External Token is charged on exit from alicenet. Each deposit must emit an event with details that uniquely describe the ERC transfer, and a unique nonce must also be included in the deposit event for utxo formation.

#### Withdrawing ERC Tokens

All withdraw requests to the bridge pools will originate from this contract, and the hash of the proofs used for the withdraw will be recorded in this contract to prevent multiple withdraws with the same burn proof.

#### Fail Safe

since all user interactions with the bridges go through this contract, we will maintain a circuit breaker here such that we can stop all deposits and withdraws on pools implementing a specific version, of bridge pool logic. To save gas on state reads, we can incorporate the lockup logic into the fees since each pool logic type will have its own fees. A pool is locked if the fee is set to max uint256 value

#### Data

<!-- Data Models / Schemas Requirements -->

For forward compatibility with future pools a dynamic data structure should be used for input data.

```solidity

struct DepositData {
    address poolAddress;
    uint16 poolVersion;
    uint8 poolType;
    uint8 ercType;
    bytes txData;
}

struct WithdrawData{
    address poolAddress;

}
```

#### Logic

bellow is an example of a dynamic event emitter. The emitDepositEvent function takes in a bytes32 array of topics, every event must have atleast one topic
can, events can have up to 4 bytes32 topics, and variables are abi encoded byte strings.

```solidity
function _emitDepositEvent(bytes32[] memory topics_, bytes memory eventData_) internal {
        if (topics_.length == 0) revert CentralBridgeRouterErrors.MissingEventSignature();
        if (topics_.length == 1) _log1Event(topics_, eventData_);
        else if (topics_.length == 2) _log2Event(topics_, eventData_);
        else if (topics_.length == 3) _log3Event(topics_, eventData_);
        else if (topics_.length == 4) _log4Event(topics_, eventData_);
        else revert CentralBridgeRouterErrors.TooManyIndexedVariables();
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
