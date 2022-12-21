# Introduction

_observations on the codebase, listing of which contracts/functions would benefit most from optimisation_

# Summary

| ID  | Finding | Instances |
| --- | ------- | --------- |
| G-01    |         |           |

# Gas Optimization Findings

## \[G-01\] Cache access of mappings and arrays / use storage pointer

When referencing the same mapping in a loop, a simple optimization is to cache access of mappings and arrays.

Additionally, in the example below, since the struct needs to be modified in storage, the `accountAssetVaultInfo` variable must be marked as `storage`.

Note that performing this optimization must be measured on a case-by-case basis, e.g. performing the same changes to `setAllowedCollateral` results in an increase of 30 gas.

### Example

Before
```solidity
src/PaprController.sol
456:     function _increaseDebt(
457:         address account,
458:         ERC721 asset,
459:         address mintTo,
460:         uint256 amount,
461:         ReservoirOracleUnderwriter.OracleInfo memory oracleInfo
462:     ) internal {
463:         uint256 cachedTarget = updateTarget();
464: 
465:         uint256 newDebt = _vaultInfo[account][asset].debt + amount;
466:         uint256 oraclePrice =
467:             underwritePriceForCollateral(asset, ReservoirOracleUnderwriter.PriceKind.LOWER, oracleInfo);
468: 
469:         uint256 max = _maxDebt(_vaultInfo[account][asset].count * oraclePrice, cachedTarget);
470: 
471:         if (newDebt > max) revert IPaprController.ExceedsMaxDebt(newDebt, max);
472: 
473:         if (newDebt >= 1 << 200) revert IPaprController.DebtAmountExceedsUint200();
474: 
475:         _vaultInfo[account][asset].debt = uint200(newDebt);
476:         PaprToken(address(papr)).mint(mintTo, amount);
477: 
478:         emit IncreaseDebt(account, asset, amount);
479:     }
```

After:
- Deployment:   🟢 13,018 gas savings
- Transaction:  🟢 325 gas savings
```solidity
src/PaprController.sol
456:     function _increaseDebt(
457:         address account,
458:         ERC721 asset,
459:         address mintTo,
460:         uint256 amount,
461:         ReservoirOracleUnderwriter.OracleInfo memory oracleInfo
462:     ) internal {
463: 
464:         VaultInfo storage accountAssetVaultInfo = _vaultInfo[account][asset];
465:         uint256 cachedTarget = updateTarget();
466: 
467:         uint256 newDebt = accountAssetVaultInfo.debt + amount;
468:         uint256 oraclePrice =
469:             underwritePriceForCollateral(asset, ReservoirOracleUnderwriter.PriceKind.LOWER, oracleInfo);
470: 
471:         uint256 max = _maxDebt(accountAssetVaultInfo.count * oraclePrice, cachedTarget);
472: 
473:         if (newDebt > max) revert IPaprController.ExceedsMaxDebt(newDebt, max);
474: 
475:         if (newDebt >= 1 << 200) revert IPaprController.DebtAmountExceedsUint200();
476: 
477:         accountAssetVaultInfo.debt = uint200(newDebt);
478:         PaprToken(address(papr)).mint(mintTo, amount);
479: 
480:         emit IncreaseDebt(account, asset, amount);
481:     }
```


---

## \[G-02\] Use named return variables

While the code already makes extensive use of named return variables, I did identify 1 more instance that saves a small amount of gas.

### 1st instance

Before:

```solidity
src/PaprController.sol
556:     function _maxDebt(uint256 totalCollateraValue, uint256 cachedTarget) internal view returns (uint256) {
557:         uint256 maxLoanUnderlying = totalCollateraValue * maxLTV;
558:         return maxLoanUnderlying / cachedTarget;
559:     }
```

After:
- Deployment:   🟢 400 gas savings
- Transaction:  🟢 7 gas savings
```solidity
src/PaprController.sol
556:     function _maxDebt(uint256 totalCollateraValue, uint256 cachedTarget) internal view returns (uint256 maxDebt) {
557:         maxDebt = totalCollateraValue * maxLTV / cachedTarget;
558:     }
```

### Not recommended

Before:
```solidity
src/NFTEDA/extensions/NFTEDAStarterIncentive.sol
53:     function _auctionCurrentPrice(uint256 id, uint256 startTime, INFTEDA.Auction memory auction)
54:         internal
55:         view
56:         virtual
57:         override
58:         returns (uint256)
59:     {
60:         uint256 price = super._auctionCurrentPrice(id, startTime, auction);
61: 
62:         if (msg.sender == auctionState[id].starter) {
63:             price = FixedPointMathLib.mulWadUp(price, _pricePercentAfterDiscount);
64:         }
65: 
66:         return price;
67:     }
```

After:
- Deployment:   🔴 1,400 gas extra
- Transaction:  🟢 2 gas savings but 1,400 saved in `testEmitsSetCreatorDiscount()`.
```solidity
src/NFTEDA/extensions/NFTEDAStarterIncentive.sol
53:     function _auctionCurrentPrice(uint256 id, uint256 startTime, INFTEDA.Auction memory auction)
54:         internal
55:         view
56:         virtual
57:         override
58:         returns (uint256)
59:     {
60:         uint256 price = super._auctionCurrentPrice(id, startTime, auction);
61: 
62:         if (msg.sender == auctionState[id].starter) {
63:             price = FixedPointMathLib.mulWadUp(price, _pricePercentAfterDiscount);
64:         }
65: 
66:         return price;
67:     }
```

---

## \[G-03] Redundant return value

In the following code, `twat` is already a named return variable and does not need to be explicitly returned.

### Example

Before:

```solidity
src/libraries/OracleLibrary.sol
34:     function timeWeightedAverageTick(int56 startTick, int56 endTick, int56 twapDuration)
35:         internal
36:         view
37:         returns (int24 twat)
38:     {
39:         require(twapDuration != 0, "BP");
40: 
41:         unchecked {
42:             int56 delta = endTick - startTick;
43: 
44:             twat = int24(delta / twapDuration);
45: 
46:             // Always round to negative infinity
47:             if (delta < 0 && (delta % (twapDuration) != 0)) {
48:                 twat--;
49:             }
50: 
51:             twat; // <-- can remove this
52:         }
53:     }
```

After:
- Transaction:  🟢 44 average gas savings


---

## \[G-04] Use `unchecked` on hot paths where variables can't overflow

Before:
Line 419 cannot practically overflow without running out of storage.

```solidity
src/PaprController.sol
418:         collateralOwner[collateral.addr][collateral.id] = account;
419:         _vaultInfo[account][collateral.addr].count += 1;
```

After:
- Deployment: 🟢 15,027 gas savings (!)
- Transaction:  🟢 273 average gas savings

```solidity
src/PaprController.sol
418:         collateralOwner[collateral.addr][collateral.id] = account;
419:         unchecked {
420:           _vaultInfo[account][collateral.addr].count += 1;
421:         }
```


---

## \[G-05] Use `++` instead of `+= 1`

This costs a bit more gas to deploy, but balances out after `600/14 = 43` operations.

Further, when combined with the previous `unchecked` optimization, the deployment cost is significantly less.

Before:

```solidity
src/PaprController.sol
419:         _vaultInfo[account][collateral.addr].count += 1;
```

After:
- Deployment: 🔴 600 gas extra cost
- Transaction:  🟢 14 average gas savings

```solidity
src/PaprController.sol
419:         ++_vaultInfo[account][collateral.addr].count;
```
