# [N-01] Different collateralArr restriction between addCollateral and removeCollateral parameter

In `PaprController.sol` function `addCollateral`, the `collateralArr` elements may contains different nft contract address, meanwhile the `removeCollateral` only check the first element of `collateralArr` , if the second element is different than the first one, it will revert.

```solidity
File: PaprController.sol
118:         for (uint256 i = 0; i < collateralArr.length;) {
119:             if (i == 0) {
120:                 collateralAddr = collateralArr[i].addr;
121:                 oraclePrice =
122:                     underwritePriceForCollateral(collateralAddr, ReservoirOracleUnderwriter.PriceKind.LOWER, oracleInfo);
123:             } else {
124:                 if (collateralAddr != collateralArr[i].addr) {
125:                     revert CollateralAddressesDoNotMatch();
126:                 }
127:             }
128: 
...
134:         }
```

This is not an error or issue, but seems an inconvenience and inconsistency. It will also save gas, if multiple nft token with different contract are trying to be removed.

# [N-02] UNSPECIFIC COMPILER VERSION PRAGMA

```solidity
pragma solidity ^0.8.17;
```

Avoid floating pragmas for non-library contracts.

While floating pragmas make sense for libraries to allow them to be included with multiple different versions of applications, it may be a security risk for application implementations.

A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain.

It is recommended to pin to a concrete compiler version.

# [N-03] No emit for storage variable change set

When changing storage variable commonly a setter function it's recommended to emit an event, meanwhile the protocol most of the setter function didn't emit an event. Emitting events allows monitoring activities with off-chain monitoring tools.

For example:

```solidity
File: PaprController.sol
360:     function setLiquidationsLocked(bool locked) external override onlyOwner {
361:         liquidationsLocked = locked;
362:     }
```