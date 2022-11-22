---
title: Bridge Pool Factory
author: z-j-lin
discussion: <!--- Discussion URL -->
team: <!--- Backend or Frontend -->
category: <!--- Development, Informational, User Experience -->
status: <!--- Draft, Review, or Final -->
related: <!-- Issue or Spec [URL] -->
created: 2022-11-21
---

## Introduction

#### Summary

This spec will discuss the workings of BridgePoolFactory.sol a thin proxy bridge pool deployer contract

#### Context

We will use [EIP-1167](https://eips.ethereum.org/EIPS/eip-1167) minimal thin proxy to clone Bridge Pool logic across multiple ERC token bridges

#### Goals

- Deploy Bridge pools at deterministic locations
- Maintain a versioned migration upgrade model. ie pool logic cannot be upgraded once deployed but new versions of pools can be deployed at a different location.
- Deploy ERC20, ERC721, and ERC1155 native and external pools

#### Non Goals

<!--- What is not to be included with this -->

#### Assumptions

- only AliceNet Factory can deploy bridge pools initially
- Bridge pool factory can be upgradeable
- We eventually want to allow the public to deploy bridge pools

## Specification

#### Overview

The bridge pool factory is a contract factory that deploys non-upgradeable bridge pool contracts at deterministic locations determined by the salt and the contracts initialization code. To avoid having to store the initialization hash of ever bridge pool contract, we will use a metamorphic contract to deploy bridge pools such that the only difference between each deployment is the salt, and we only need to store one initcode hash, bridgePool factory address, and salt to calculate the address of a bridge pool.

Each ERC token will have a bridge pool contract that either is a wrapper contract for a external token or a holder contract for a native token. Since bridge pool contracts hold value, they must not be upgradeable. Since multiple pools with the same logic exists, we will deploy the logic for the bridge pools once and each ERC token bridge will be deployed with a EIP-1167 minimal thin proxy contract for each ERC token bridge we will deploy. For upgradeability new pool logic can be deployed and new ERC bridge pools can be deployed to point at new pool logic.

since bridge pools cannot be upgraded or destroyed, and the address of each pool is determined by the salt. Salt used to deploy bridge pool deployment must be in a format that is collision resistant such that multiple versions of bridge pools representing the same ERC token can be deployed.

#### Data

Mappings
deployed bridge pool logics can be referenced with a unique key string from the \_logicAddresses mapping

```solidity
/**
     * @notice calculates salt for a BridgePool implementation contract based on tokenType and version
     * @param tokenType_ type of token (0=ERC20, 1=ERC721, 2=ERC1155)
     * @param version_ version of the implementation
     * @param native_ boolean flag to specifier native or external token pools
     * @return calculated key
     */
    function _getImplementationAddressKey(
        uint8 tokenType_,
        uint16 version_,
        bool native_
    ) internal pure returns (string memory) {
        string memory key;
        if (native_) {
            key = "Native";
        } else {
            key = "External";
        }
        if (tokenType_ == uint8(TokenType.ERC20)) {
            key = string.concat(key, "ERC20");
        } else if (tokenType_ == uint8(TokenType.ERC721)) {
            key = string.concat(key, "ERC721");
        } else if (tokenType_ == uint8(TokenType.ERC1155)) {
            key = string.concat(key, "ERC1155");
        }
        key = string.concat(key, "V", Strings.toString(version_));
        return key;
    }
```

mapping(string => address) internal \_logicAddresses;

##### Bridge pool salt calculation

#### Logic

Metamorphic Contract V2

This metamorphic contract is intended to be used with a factory contract using create2 contract deployment. When deployed through a contract, while it is in the constructor phase of deployment, static call back to the caller with no call data, and hits the fallback function of the deployer contract. At the completion of the constructor phase this contract will write everything in return data to the runtime code section.
0x5880818283335afa3d82833e3d82f3

| PC  |     OPCODE     | STACK                        | YUL                                 | inputs                                                 | output        |
| :-- | :------------: | :--------------------------- | :---------------------------------- | :----------------------------------------------------- | :------------ |
| 0   |       PC       |                              |                                     |                                                        | PC            |
| 1   |      DUP1      | 0                            |                                     |                                                        |               |
| 2   |      DUP2      | 0, 0                         |                                     |                                                        |               |
| 3   |      DUP3      | 0, 0, 0                      |                                     |                                                        |               |
| 4   |      DUP4      | 0, 0, 0, 0                   |                                     |                                                        |               |
| 5   |     CALLER     | 0, 0, 0, 0, 0                |                                     |                                                        | address       |
| 6   |      GAS       | CALLER, 0, 0, 0, 0, 0        |                                     |                                                        | gas           |
| 7   |   STATICCALL   | GAS, CALLER, 0, 0, 0, 0, 0   | staticcall(GAS, CALLER, 0, 0, 0, 0) | gas, address, argsOffset, argsSize, retOffset, retSize | success(bool) |
| 8   | RETURNDATASIZE | 0/1, 0                       | returndatasize()                    |                                                        | size          |
| 9   |      DUP3      | RETURNDATASIZE, 0/1, 0       |                                     |                                                        |               |
| 10  |      DUP4      | 0, RETURNDATASIZE, 0/1, 0    |                                     |                                                        |               |
| 11  | RETURNDATACOPY | 0, 0, RETURNDATASIZE, 0/1, 0 |                                     | destOffset, offset, size                               |               |
| 12  | RETURNDATASIZE | 1, 0                         |                                     |                                                        | size          |
| 13  |      DUP3      | RETURNDATASIZE, 1, 0         |                                     |                                                        |               |
| 14  |     RETURN     | 0, RETURNDATASIZE, 1, 0      |                                     |                                                        |               |

Example fallback function for above metamorphic contract

```solidity
/**
     * @notice returns bytecode for a Minimal Proxy (EIP-1167) that routes to BridgePool implementation
     */
    // solhint-disable-next-line
    fallback() external {
        address implementation_ = _implementation;
        assembly {
            let ptr := mload(0x40)
            mstore(ptr, shl(176, 0x363d3d373d3d3d363d73)) //10
            mstore(add(ptr, 10), shl(96, implementation_)) //20
            mstore(add(ptr, 30), shl(136, 0x5af43d82803e903d91602b57fd5bf3)) //15
            return(ptr, 45)
        }
    }
```

This is the EIP-1167 thin proxy bytecode that will be written to runtime code of bridge pool contracts.
This contract pushes the implementation address to the stack as an immutable address and delegateCalls to the implementation contract with everything in calldata and then returns all data returned by this call.
0x363d3d373d3d3d363d73<\ImplementationAddresss>5af43d82803e903d91602b57fd5bf3

| PC  |     OPCODE     | STACK                                                              | YUL                                              |
| :-- | :------------: | :----------------------------------------------------------------- | :----------------------------------------------- |
| 00  |  CALLDATASIZE  |                                                                    |                                                  |
| 01  | RETURNDATASIZE | CALLDATASIZE                                                       |                                                  |
| 02  | RETURNDATASIZE | 0, CALLDATASIZE                                                    |                                                  |
| 03  |  CALLDATACOPY  | 0, 0, CALLDATASIZE                                                 |                                                  |
| 04  | RETURNDATASIZE |                                                                    |                                                  |
| 05  | RETURNDATASIZE | 0                                                                  |                                                  |
| 06  | RETURNDATASIZE | 0, 0                                                               |                                                  |
| 07  |  CALLDATASIZE  | 0, 0, 0                                                            |                                                  |
| 08  | RETURNDATASIZE | CALLDATASIZE, 0, 0, 0                                              | returndatasize()                                 |
| 09  |     PUSH20     | RETURNDATASIZE, CALLDATASIZE, 0, 0, 0                              |                                                  |
| 1e  |      GAS       | ImplementationAddresss, RETURNDATASIZE, CALLDATASIZE, 0, 0, 0      |                                                  |
| 1f  |  DELEGATECALL  | GAS, ImplementationAddresss, RETURNDATASIZE, CALLDATASIZE, 0, 0, 0 | delegatecall(GAS,Addresss, RDSIZE, CDSIZE, 0, 0) |
| 20  | RETURNDATASIZE | 0/1, 0                                                             |                                                  |
| 21  |      DUP3      | RETURNDATASIZE, 0/1, 0                                             |                                                  |
| 22  |      DUP1      | 0, RETURNDATASIZE, 0/1, 0                                          |                                                  |
| 23  | RETURNDATACOPY | 0, 0, RETURNDATASIZE, 0/1, 0                                       |                                                  |
| 24  |     SWAP1      | 0/1, 0                                                             |                                                  |
| 25  | RETURNDATASIZE | 0, 0/1                                                             |                                                  |
| 26  |     SWAP2      | RETURNDATASIZE, 0, 0/1                                             |                                                  |
| 27  |     PUSH1      | 0/1, 0, RETURNDATASIZE                                             |                                                  |
| 29  |     JUMPI      | 2b, 0/1, 0, RETURNDATASIZE                                         |                                                  |
| 2a  |     REVERT     |                                                                    |                                                  |
| 2b  |    JUMPDEST    | 0, RETURNDATASIZE                                                  |                                                  |
| 2c  |     RETURN     | 0, RETURNDATASIZE                                                  |                                                  |

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
