
## G-01 x += y COSTS MORE GAS THAN x = x + y FOR STATE VARIABLES (SAME APPLIES FOR x -= y , x = x - y)

_There are 2 instances of this issue_

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol

```
File: src/PaprController.sol

326: info.count -= 1;
419: _vaultInfo[account][collateral.addr].count += 1
```

-----

## G-02 USAGE OF UINTS OR INTS SMALLER THAN 32 BYTES INCURS OVERHEAD

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

[https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html](https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html) Use a larger size then downcast where needed

_There are 11 instances of this issue_

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol

```
File: src/PaprController.sol

83: uint160 initSqrtRatio
436: uint16 newCount;
```

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol

```
File: src/UniswapOracleFundingRateController.sol

29: uint128 internal _target;
31: uint48 internal _lastUpdated;
```

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/OracleLibrary.sol

```
File: src/libraries/OracleLibrary.sol

10: function getQuoteAtTick(int24 tick, uint128 baseAmount, address baseToken, address quoteToken)
16: uint160 sqrtRatioX96 = TickMath.getSqrtRatioAtTick(tick)
57: uint32[] memory secondAgos = new uint32[](1);
```

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/PoolAddress.sol

```
File: src/libraries/PoolAddress.sol

22: function getPoolKey(address tokenA, address tokenB, uint24 fee) internal pure returns (PoolKey memory) {
```

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/UniswapHelpers.sol

```
File: src/libraries/UniswapHelpers.sol

37: uint160 sqrtPriceLimitX96,
69: function deployAndInitPool(address tokenA, address tokenB, uint24 feeTier, uint160 sqrtRatio)
```

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol

```
File: src/NFTEDA/extensions/NFTEDAStarterIncentive.sol

21: event SetAuctionCreatorDiscount(uint256 discount);
```

---------

## G-03 INTERNAL FUNCTIONS ONLY CALLED ONCE CAN BE INLINED TO SAVE GAS

Not inlining costs 20 to 40 gas because of two extra `JUMP` instructions and additional stack operations needed for function calls.

_There are 16 instances of this issue_

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol

```
File: src/PaprController.sol

424: function _removeCollateral(
493: function _increaseDebtAndSell(
519: function _purchaseNFTAndUpdateVaultIfNeeded(Auction calldata auction, uint256 maxPrice, address sendTo)
532: function _handleExcess(uint256 excess, uint256 neededToSaveVault, uint256 debtCached, Auction calldata auction)
```

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol

```
File: src/UniswapOracleFundingRateController.sol

156: function _multiplier(uint256 _mark_, uint256 cachedTarget) internal view returns (uint256) {
```

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/OracleLibrary.sol

```
File: src/libraries/OracleLibrary.sol

10: function getQuoteAtTick(int24 tick, uint128 baseAmount, address baseToken, address quoteToken)
34: function timeWeightedAverageTick(int56 startTick, int56 endTick, int56 twapDuration)
```

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/PoolAddress.sol

```
File: src/libraries/PoolAddress.sol

22: function getPoolKey(address tokenA, address tokenB, uint24 fee) internal pure returns (PoolKey memory) {
31: function computeAddress(address factory, PoolKey memory key) internal pure returns (address pool) {
```

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/UniswapHelpers.sol

```
File: src/libraries/UniswapHelpers.sol

69: function deployAndInitPool(address tokenA, address tokenB, uint24 feeTier, uint160 sqrtRatio)
82: function poolCurrentTick(address pool) internal returns (int24) {
92: function poolsHaveSameTokens(address pool1, address pool2) internal view returns (bool) {
100: function isUniswapPool(address pool) internal view returns (bool) {
```

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/NFTEDA.sol

```
File: src/NFTEDA/NFTEDA.sol

48: function _startAuction(INFTEDA.Auction memory auction) internal virtual returns (uint256 id) {
72: function _purchaseNFT(INFTEDA.Auction memory auction, uint256 maxPrice, address sendTo)
```

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol

```
File: src/NFTEDA/extensions/NFTEDAStarterIncentive.sol

72: function _setCreatorDiscount(uint256 _auctionCreatorDiscountPercentWad) internal virtual {
```

-------

## G-04 MULTIPLICATION OR DIVISION BY 2 SHOULD USE BIT SHIFTING

`<x> * 2` is equivalent to `<x> << 1` and `<x> / 2` is the same as `<x> >> 1`. The `MUL` and `DIV` opcodes cost 5 gas, whereas `SHL` and `SHR` only cost 3 gas

_There is 1 instance of this issue_

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/UniswapHelpers.sol

```
File: src/libraries/UniswapHelpers.sol

111: return TickMath.getSqrtRatioAtTick(TickMath.getTickAtSqrtRatio(uint160((token1ONE << 96) / token0ONE)) / 2);
```

-----
