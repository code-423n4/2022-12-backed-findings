# Report

## Gas Optimisation

| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | USAGE OF UINT/INT SMALLER THAN 32 BYTES (256 BITS) INCURS OVERHEAD | 4 |
| [GAS-2](#GAS-2) | <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES | 2 |
| [GAS-3](#GAS-3) | USING BOOLS FOR STORAGE INCURS OVERHEAD | 1 |
| [GAS-4](#GAS-4) | ABI.ENCODE() IS LESS EFFICIENT THAN ABI.ENCODEPACKED() | 7 |
| [GAS-5](#GAS-5) | PREFIX DECREMENTS CHEAPER THAN POSTFIX DECREMENTS | 1 |

###  [GAS-1] USAGE OF UINT/INT SMALLER THAN 32 BYTES (256 BITS) INCURS OVERHEAD
When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.
https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html
Use a larger size then downcast where needed.

*Instances (4)*:
[Instance 1](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/interfaces/IPaprController.sol#L37)
[Instance 2](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/interfaces/IPaprController.sol#L59)
[Instance 3](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/PoolAddress.sol#L14)
[Instance 4](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol#L12)


###  [GAS-2] <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES
Using the addition operator instead of plus-equals saves gas.

*Instances (2)*:
```solidity
File: src/PaprController.sol

326:             info.count -= 1;

419:             _vaultInfo[account][collateral.addr].count += 1;

```
[Link to code](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L326)

###  [GAS-3] USING BOOLS FOR STORAGE INCURS OVERHEAD
Booleans are more expensive than uint256 or any type that takes up a full word because each write operation emits an extra SLOAD to first read the slot’s contents, replace the bits taken up by the boolean, and then write back. This is the compiler’s defense against contract upgrades and pointer aliasing, and it cannot be disabled. Use uint256(1) and uint256(2) for true/false instead

*Instances (1)*:
```solidity
File: src/PaprController.sol

60:             mapping(address => bool) public override isAllowed;

```
[Link to code](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#60)

###  [GAS-4] ABI.ENCODE() IS LESS EFFICIENT THAN ABI.ENCODEPACKED()
use abi.encodePacked() where possible to save gas

*Instances (7)*:
```solidity
File: src/PaprController.sol

197:             abi.encode(msg.sender, collateralAsset, oracleInfo)

222:             abi.encode(msg.sender)

509:             abi.encode(account, collateralAsset, oracleInfo)
```
[Link to code](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#197)
```solidity
File: src/ReservoirOracleUnderwriter.sol

74:             abi.encode(

93:             abi.encode(

```
[Link to code](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/ReservoirOracleUnderwriter.sol#74)

```solidity
File: src/libraries/PoolAddress.sol

40:             keccak256(abi.encode(key.token0, key.token1, key.fee)),

```
[Link to code](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/PoolAddress.sol#40)

```solidity
File: src/NFTEDA/NFTEDA.sol

36:             return uint256(keccak256(abi.encode(auction)));

```
[Link to code](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/NFTEDA.sol#36)

###  [GAS-5] PREFIX DECREMENTS CHEAPER THAN POSTFIX DECREMENTS
twat-- can be replaced with --twat;
*Instances (1)*:
```solidity
File: src/libraries/OracleLibrary.sol

48:             twat--;

```
[Link to code](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/OracleLibrary.sol#48)
