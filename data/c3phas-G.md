## FINDINGS
NB: Some functions have been truncated where necessary to just show affected parts of the code
Throughout the report some places might be denoted with audit tags to show the actual place affected.

## Table of contents
- [FINDINGS](#findings)
- [Note on Gas estimates.](#note-on-gas-estimates)
- [Internal/Private functions only called once can be inlined to save gas](#internalprivate-functions-only-called-once-can-be-inlined-to-save-gas)
- [Multiple accesses of a mapping/array should use a local variable cache](#multiple-accesses-of-a-mappingarray-should-use-a-local-variable-cache)
  - [PaprController.sol.purchaseLiquidationAuctionNFT(): \_vaultInfo\[auction.nftOwner\]\[auction.auctionAssetContract\] should be cached in local storage](#paprcontrollersolpurchaseliquidationauctionnft-_vaultinfoauctionnftownerauctionauctionassetcontract-should-be-cached-in-local-storage)
  - [PaprController.sol.\_removeCollateral(): \_vaultInfo\[msg.sender\]\[collateral.addr\] should be cached in local storage](#paprcontrollersol_removecollateral-_vaultinfomsgsendercollateraladdr-should-be-cached-in-local-storage)
  - [PaprController.sol.\_increaseDebt(): \_vaultInfo\[account\]\[asset\] should be cached in local storage](#paprcontrollersol_increasedebt-_vaultinfoaccountasset-should-be-cached-in-local-storage)
  - [PaprController.sol.\_purchaseNFTAndUpdateVaultIfNeeded(): \_vaultInfo\[auction.nftOwner\]\[auction.auctionAssetContract\] should be cached in local storage](#paprcontrollersol_purchasenftandupdatevaultifneeded-_vaultinfoauctionnftownerauctionauctionassetcontract-should-be-cached-in-local-storage)
- [Avoid contract existence checks by using solidity version 0.8.10 or later](#avoid-contract-existence-checks-by-using-solidity-version-0810-or-later)
- [x += y costs more gas than x = x + y for state variables](#x--y-costs-more-gas-than-x--x--y-for-state-variables)
- [Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead](#usage-of-uintsints-smaller-than-32-bytes-256-bits-incurs-overhead)
- [Using unchecked blocks to save gas](#using-unchecked-blocks-to-save-gas)
- [`keccak256()` should only need to be called on a specific string literal once](#keccak256-should-only-need-to-be-called-on-a-specific-string-literal-once)
- [Functions guaranteed to revert when called by normal users can be marked `payable`](#functions-guaranteed-to-revert-when-called-by-normal-users-can-be-marked-payable)
- [The following should have been caught by the c4udit but wasn't, adding for completeness](#the-following-should-have-been-caught-by-the-c4udit-but-wasnt-adding-for-completeness)
  - [Cache storage values in memory to minimize SLOADs](#cache-storage-values-in-memory-to-minimize-sloads)
  - [UniswapOracleFundingRateController.sol.\_setPool(): pool should be cached](#uniswaporaclefundingratecontrollersol_setpool-pool-should-be-cached)

## Note on Gas estimates.
I've tried to give the exact amount of gas being saved from running the included tests. 

Some functions are not covered by the test cases or are internal/private functions. 


## Internal/Private functions only called once can be inlined to save gas

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

Affected code:
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L424-L429
```solidity
File: /src/PaprController.sol
424:    function _removeCollateral(
425:        address sendTo,
426:        IPaprController.Collateral calldata collateral,
427:        uint256 oraclePrice,
428:        uint256 cachedTarget
429:    ) internal {
```

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L493-L517
```solidity
File: /src/PaprController.sol
493:    function _increaseDebtAndSell(
494:        address account,
495:        address proceedsTo,
496:        ERC721 collateralAsset,
497:        IPaprController.SwapParams memory params,
498:        ReservoirOracleUnderwriter.OracleInfo memory oracleInfo
499:    ) internal returns (uint256 amountOut) {
```

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L519-L530
```solidity
File: /src/PaprController.sol
519:    function _purchaseNFTAndUpdateVaultIfNeeded(Auction calldata auction, uint256 maxPrice, address sendTo)
520:        internal
521:        returns (uint256)
522:    {
```

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L532-L552
```solidity
File: /src/PaprController.sol
532:    function _handleExcess(uint256 excess, uint256 neededToSaveVault, uint256 debtCached, Auction calldata auction)
533:        internal
534:        returns (uint256 remaining)
535:    {
```

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L95
```solidity
File:/src/UniswapOracleFundingRateController.sol
95:    function _init(uint256 target, address _pool) internal {

```

## Multiple accesses of a mapping/array should use a local variable cache

Caching a mapping's value in a local storage or calldata variable when the value is accessed multiple times saves ~42 gas per access due to not having to perform the same offset calculation every time.
**Help the Optimizer by saving a storage variable's reference instead of repeatedly fetching it**

To help the optimizer,declare a storage type variable and use it instead of repeatedly fetching the reference in a map or an array. 
As an example, instead of repeatedly calling ```someMap[someIndex]```, save its reference like this: ```SomeStruct storage someStruct = someMap[someIndex]``` and use it.

**see audit tags for the occurence**

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L264-L294
### PaprController.sol.purchaseLiquidationAuctionNFT(): \_vaultInfo\[auction.nftOwner]\[auction.auctionAssetContract] should be cached in local storage
```solidity
File: /src/PaprController.sol
264:    function purchaseLiquidationAuctionNFT(

270:        uint256 collateralValueCached = underwritePriceForCollateral(
271:            auction.auctionAssetContract, ReservoirOracleUnderwriter.PriceKind.TWAP, oracleInfo
272:        ) * _vaultInfo[auction.nftOwner][auction.auctionAssetContract].count;//@audit: 1st access
273:        bool isLastCollateral = collateralValueCached == 0;

275:        uint256 debtCached = _vaultInfo[auction.nftOwner][auction.auctionAssetContract].debt;//@audit: 2nd access

294:    }
```

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L424-L454
### PaprController.sol.\_removeCollateral(): \_vaultInfo\[msg.sender]\[collateral.addr] should be cached in local storage
```solidity
File: /src/PaprController.sol
424:    function _removeCollateral(

437:        unchecked {
438:            newCount = _vaultInfo[msg.sender][collateral.addr].count - 1;//@audit: Initial access
439:            _vaultInfo[msg.sender][collateral.addr].count = newCount;//@audit: 2nd access
440:        }

446:        uint256 debt = _vaultInfo[msg.sender][collateral.addr].debt; //@audit: 3rd access
447:        uint256 max = _maxDebt(oraclePrice * newCount, cachedTarget);
```

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L456-L479
### PaprController.sol.\_increaseDebt(): \_vaultInfo\[account]\[asset] should be cached in local storage
```solidity
File: /src/PaprController.sol
456:    function _increaseDebt(

465:        uint256 newDebt = _vaultInfo[account][asset].debt + amount; //@audit: 1st access

469:        uint256 max = _maxDebt(_vaultInfo[account][asset].count * oraclePrice, cachedTarget);//@audit: 2nd access

475:        _vaultInfo[account][asset].debt = uint200(newDebt); //@audit: 3rd access
```

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L519-L530
### PaprController.sol.\_purchaseNFTAndUpdateVaultIfNeeded(): \_vaultInfo\[auction.nftOwner]\[auction.auctionAssetContract] should be cached in local storage
```solidity
File: /src/PaprController.sol
519:    function _purchaseNFTAndUpdateVaultIfNeeded(Auction calldata auction, uint256 maxPrice, address sendTo)


525:        if (startTime == _vaultInfo[auction.nftOwner][auction.auctionAssetContract].latestAuctionStartTime) { //@audit: Initial access
526:            _vaultInfo[auction.nftOwner][auction.auctionAssetContract].latestAuctionStartTime = 0; //@audit: 2nd access
527:        }
```

## Avoid contract existence checks by using solidity version 0.8.10 or later

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value

**Total Instances:** 7
`Gas savings: 7 * 100 = 700 gas`

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/OracleLibrary.sol#L59
```solidity
File: /src/libraries/OracleLibrary.sol

//@audit: observe()
59:        (int56[] memory tickCumulatives,) = IUniswapV3Pool(pool).observe(secondAgos);
```

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/UniswapHelpers.sol#L40-L48
```solidity
File: /src/libraries/UniswapHelpers.sol

//@audit: swap()
40:        (int256 amount0, int256 amount1) = IUniswapV3Pool(pool).swap(
41:            recipient,
42:            zeroForOne,
43:            amountSpecified.toInt256(),
44:            sqrtPriceLimitX96 == 0
45:                ? (zeroForOne ? TickMath.MIN_SQRT_RATIO + 1 : TickMath.MAX_SQRT_RATIO - 1)
46:                : sqrtPriceLimitX96,
47:            data
48:        );

//@audit: createPool()
73:        IUniswapV3Pool pool = IUniswapV3Pool(FACTORY.createPool(tokenA, tokenB, feeTier));

//@audit: slot0()
83:        (, int24 tick,,,,,) = IUniswapV3Pool(pool).slot0();

//@audit: token0(), token1()
93:        return IUniswapV3Pool(pool1).token0() == IUniswapV3Pool(pool2).token0()
94:            && IUniswapV3Pool(pool1).token1() == IUniswapV3Pool(pool2).token1();

//@audit: token0(), token1()
102:        PoolAddress.PoolKey memory k = PoolAddress.getPoolKey(p.token0(), p.token1(), p.fee());
```

## x += y costs more gas than x = x + y for state variables

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L326
**Save 42 gas on average**

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 973    | 47311   | 42708 | 75484 |
| After  | 973    | 47269   | 42657 | 75421 |

```solidity
File: /src/PaprController.sol
326:        info.count -= 1;
```

```diff
diff --git a/src/PaprController.sol b/src/PaprController.sol
index 284b3f4..5b856ca 100644
--- a/src/PaprController.sol
+++ b/src/PaprController.sol
@@ -323,7 +323,7 @@ contract PaprController is
         }

         info.latestAuctionStartTime = uint40(block.timestamp);
-        info.count -= 1;
+        info.count = info.count - 1;
```

## Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html Use a larger size then downcast where needed

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L436
```solidity
File: /src/PaprController.sol

//@audit: newCount 
436:        uint16 newCount;
```

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/PoolAddress.sol#L22
```solidity
File: /src/libraries/PoolAddress.sol

//@audit: uint24 fee
22:    function getPoolKey(address tokenA, address tokenB, uint24 fee) internal pure returns (PoolKey memory) {
```

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/OracleLibrary.sol#L10
```solidity
File: /src/libraries/OracleLibrary.sol

//@audit: int24 tick,uint128 baseAmount
10:    function getQuoteAtTick(int24 tick, uint128 baseAmount, address baseToken, address quoteToken)

//@audit: uint160 sqrtRatioX96
16:            uint160 sqrtRatioX96 = TickMath.getSqrtRatioAtTick(tick);

//@audit: int56 startTick,int56 endTick,int56 twapDuration
34:    function timeWeightedAverageTick(int56 startTick, int56 endTick, int56 twapDuration)

//@audit:int56 delta 
42:            int56 delta = endTick - startTick;
```

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/UniswapHelpers.sol#L31-L39
```solidity
File: /src/libraries/UniswapHelpers.sol

//@audit: uint160 sqrtPriceLimitX96
31:    function swap(
32:        address pool,
33:        address recipient,
34:        bool zeroForOne,
35:        uint256 amountSpecified,
36:        uint256 minOut,
37:        uint160 sqrtPriceLimitX96,
38:        bytes memory data
39:    ) internal returns (uint256 amountOut, uint256 amountIn) {

//@audit: uint24 feeTier, uint160 sqrtRatio
69:    function deployAndInitPool(address tokenA, address tokenB, uint24 feeTier, uint160 sqrtRatio)
```

## Using unchecked blocks to save gas
Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block
[see resource](https://github.com/ethereum/solidity/issues/10695)

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L278
**Saves 46 gas on average**

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 9430    | 85051   | 89316 | 109994 |
| After  | 9430    | 85005   | 89263 | 109941 |


```solidity
File: /src/PaprController.sol
278:        uint256 neededToSaveVault = maxDebtCached > debtCached ? 0 : debtCached - maxDebtCached;
```

```diff
diff --git a/src/PaprController.sol b/src/PaprController.sol
index 284b3f4..f7f2a3c 100644
--- a/src/PaprController.sol
+++ b/src/PaprController.sol
@@ -275,7 +275,10 @@ contract PaprController is
         uint256 debtCached = _vaultInfo[auction.nftOwner][auction.auctionAssetContract].debt;
         uint256 maxDebtCached = isLastCollateral ? debtCached : _maxDebt(collateralValueCached, updateTarget());
         /// anything above what is needed to bring this vault under maxDebt is considered excess
-        uint256 neededToSaveVault = maxDebtCached > debtCached ? 0 : debtCached - maxDebtCached;
+        uint256 neededToSaveVault;
+        unchecked {
+            neededToSaveVault = maxDebtCached > debtCached ? 0 : debtCached - maxDebtCached;
+        }
```


https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L280
**Saves 39 gas on average**

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 9430    | 85051   | 89316 | 109994 |
| After  | 9430    | 85012   | 89264 | 109942 |

```solidity
File: /src/PaprController.sol
280:        uint256 excess = price > neededToSaveVault ? price - neededToSaveVault : 0;
```

```diff
diff --git a/src/PaprController.sol b/src/PaprController.sol
index 284b3f4..4e97529 100644
--- a/src/PaprController.sol
+++ b/src/PaprController.sol
@@ -277,7 +277,10 @@ contract PaprController is
         /// anything above what is needed to bring this vault under maxDebt is considered excess
         uint256 neededToSaveVault = maxDebtCached > debtCached ? 0 : debtCached - maxDebtCached;
         uint256 price = _purchaseNFTAndUpdateVaultIfNeeded(auction, maxPrice, sendTo);
-        uint256 excess = price > neededToSaveVault ? price - neededToSaveVault : 0;
+        uint256 excess;
+        unchecked {
+            excess = price > neededToSaveVault ? price - neededToSaveVault : 0;
+        }
```


https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L546
```solidity
File: /src/PaprController.sol
546:            papr.transfer(auction.nftOwner, totalOwed - debtCached);
```

The operation `totalOwed - debtCached` cannot underflow due to the check on [Line 542](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L542) that ensures that `totalOwed` is greater than `debtCached`

```diff
diff --git a/src/PaprController.sol b/src/PaprController.sol
index 284b3f4..3099a56 100644
--- a/src/PaprController.sol
+++ b/src/PaprController.sol
@@ -543,7 +543,10 @@ contract PaprController is
             // we owe them more papr than they have in debt
             // so we pay down debt and send them the rest
             _reduceDebt(auction.nftOwner, auction.auctionAssetContract, address(this), debtCached);
-            papr.transfer(auction.nftOwner, totalOwed - debtCached);
+            unchecked {
+               papr.transfer(auction.nftOwner, totalOwed - debtCached);
+            }
```


https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L550
```solidity
File: /src/PaprController.sol
550:            remaining = debtCached - totalOwed;
```

The operation `debtCached - totalOwed` would only be evaluated if `debtCached` is greater than `totalOwed` 

```diff
diff --git a/src/PaprController.sol b/src/PaprController.sol
index 284b3f4..824ce8b 100644
--- a/src/PaprController.sol
+++ b/src/PaprController.sol
@@ -547,7 +547,10 @@ contract PaprController is
         } else {
             // reduce vault debt
             _reduceDebt(auction.nftOwner, auction.auctionAssetContract, address(this), totalOwed);
-            remaining = debtCached - totalOwed;
+            unchecked {
+               remaining = debtCached - totalOwed;
+            }
         }
     }
```

## `keccak256()` should only need to be called on a specific string literal once

It should be saved to an immutable variable, and the variable used instead. If the hash is being used as a part of a function selector, the cast to `bytes4` should also only be done once

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/ReservoirOracleUnderwriter.sol#L75
```solidity
File: /src/ReservoirOracleUnderwriter.sol
75:                            keccak256("Message(bytes32 id,bytes payload,uint256 timestamp)"),
```

```diff
diff --git a/src/ReservoirOracleUnderwriter.sol b/src/ReservoirOracleUnderwriter.sol
index 195b363..770564a 100644
--- a/src/ReservoirOracleUnderwriter.sol
+++ b/src/ReservoirOracleUnderwriter.sol
@@ -43,6 +43,8 @@ contract ReservoirOracleUnderwriter {
     /// @notice address of the currency we are receiving oracle prices in
     address public immutable quoteCurrency;

+    bytes32 public immutable MESSAGE = keccak256("Message(bytes32 id,bytes payload,uint256 timestamp)");
+
     error IncorrectOracleSigner();
     error WrongIdentifierFromOracleMessage();
     error WrongCurrencyFromOracleMessage();
@@ -72,7 +74,7 @@ contract ReservoirOracleUnderwriter {
                     // EIP-712 structured-data hash
                     keccak256(
                         abi.encode(
-                            keccak256("Message(bytes32 id,bytes payload,uint256 timestamp)"),
+                            MESSAGE,
                             oracleInfo.message.id,
                             keccak256(oracleInfo.message.payload),
                             oracleInfo.message.timestamp
```

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/ReservoirOracleUnderwriter.sol#L94
```solidity
File: /src/ReservoirOracleUnderwriter.sol
94:                keccak256("ContractWideCollectionPrice(uint8 kind,uint256 twapSeconds,address contract)"),
```

```diff
diff --git a/src/ReservoirOracleUnderwriter.sol b/src/ReservoirOracleUnderwriter.sol
index 195b363..dff619e 100644
--- a/src/ReservoirOracleUnderwriter.sol
+++ b/src/ReservoirOracleUnderwriter.sol
@@ -43,6 +43,8 @@ contract ReservoirOracleUnderwriter {
     /// @notice address of the currency we are receiving oracle prices in
     address public immutable quoteCurrency;

+    bytes32 public immutable ContractWideCollectionPrice = keccak256("ContractWideCollectionPrice(uint8 kind,uint256 twapSeconds,address contract)");
+
     error IncorrectOracleSigner();
     error WrongIdentifierFromOracleMessage();
     error WrongCurrencyFromOracleMessage();
@@ -91,7 +93,7 @@ contract ReservoirOracleUnderwriter {

         bytes32 expectedId = keccak256(
             abi.encode(
-                keccak256("ContractWideCollectionPrice(uint8 kind,uint256 twapSeconds,address contract)"),
+                ContractWideCollectionPrice,
                 priceKind,
                 TWAP_SECONDS,
                 asset
```


## Functions guaranteed to revert when called by normal users can be marked `payable`

If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.The extra opcodes avoided costs an average of about **21 gas per call** to the function, in addition to the extra deployment cost

`Total Instances: 8 `
`Gas savings: 8 * 21 = 161 gas`

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L350
```solidity
File: /src/PaprController.sol
350:    function setPool(address _pool) external override onlyOwner {

355:    function setFundingPeriod(uint256 _fundingPeriod) external override onlyOwner {

360:    function setLiquidationsLocked(bool locked) external override onlyOwner {

365:    function setAllowedCollateral(IPaprController.CollateralAllowedConfig[] calldata collateralConfigs)
366:        external
367:        override
368:        onlyOwner
369:    {


382:    function sendPaprFromAuctionFees(address to, uint256 amount) external override onlyOwner {

386:    function burnPaprFromAuctionFees(uint256 amount) external override onlyOwner {
```

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprToken.sol#L24
```solidity
File: /src/PaprToken.sol
24:    function mint(address to, uint256 amount) external onlyController {

28:    function burn(address account, uint256 amount) external onlyController {
```


## The following should have been caught by the c4udit but wasn't, adding for completeness

### Cache storage values in memory to minimize SLOADs
The code can be optimized by minimizing the number of SLOADs.

SLOADs are expensive (100 gas after the 1st one) compared to MLOADs/MSTOREs (3 gas each). Storage values read multiple times should instead be cached in memory the first time (costing 1 SLOAD) and then read from this cache to avoid multiple SLOADs.

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L111-L118
### UniswapOracleFundingRateController.sol.\_setPool(): pool should be cached
```solidity
File: /src/UniswapOracleFundingRateController.sol
111:    function _setPool(address _pool) internal {
112:        if (pool != address(0) && !UniswapHelpers.poolsHaveSameTokens(pool, _pool)) revert PoolTokensDoNotMatch();
113:        if (!UniswapHelpers.isUniswapPool(_pool)) revert InvalidUniswapV3Pool();
```

```diff
diff --git a/src/UniswapOracleFundingRateController.sol b/src/UniswapOracleFundingRateController.sol
index 7179ed4..2ac3428 100644
--- a/src/UniswapOracleFundingRateController.sol
+++ b/src/UniswapOracleFundingRateController.sol
@@ -109,7 +109,8 @@ contract UniswapOracleFundingRateController is IUniswapOracleFundingRateControll
     /// @dev reverts if new pool does not have same token0 and token1 as `pool`
     /// @dev if pool = address(0), does NOT check that tokens match papr and underlying
     function _setPool(address _pool) internal {
-        if (pool != address(0) && !UniswapHelpers.poolsHaveSameTokens(pool, _pool)) revert PoolTokensDoNotMatch();
+        address _poolAddr = pool;
+        if (_poolAddr != address(0) && !UniswapHelpers.poolsHaveSameTokens(_poolAddr, _pool)) revert PoolTokensDoNotMatch();
         if (!UniswapHelpers.isUniswapPool(_pool)) revert InvalidUniswapV3Pool();
```
