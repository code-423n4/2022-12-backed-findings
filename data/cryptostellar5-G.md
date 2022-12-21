### G-01 X += Y COSTS MORE GAS THAN X = X + Y FOR STATE VARIABLES

*Number of Instances Identified: 2*

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol

```
326: info.count -= 1;
419: _vaultInfo[account][collateral.addr].count += 1;
```


### G-02 INTERNAL FUNCTIONS ONLY CALLED ONCE CAN BE INLINED TO SAVE GAS

*Number of Instances Identified: 13*

Not inlining costs **20 to 40 gas** because of two extra `JUMP` instructions and additional stack operations needed for function calls.

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol

```
424: function _removeCollateral()
493: function _increaseDebtAndSell
519: function _purchaseNFTAndUpdateVaultIfNeeded(Auction calldata auction, uint256 maxPrice, address sendTo)
532: function _handleExcess(uint256 excess, uint256 neededToSaveVault, uint256 debtCached, Auction calldata auction)
```

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol

```
156: function _multiplier(uint256 _mark_, uint256 cachedTarget)
```

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/PoolAddress.sol

```
22: function getPoolKey(address tokenA, address tokenB, uint24 fee)
31: function computeAddress(address factory, PoolKey memory key)
```

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/UniswapHelpers.sol

```
69: function deployAndInitPool(address tokenA, address tokenB, uint24 feeTier, uint160 sqrtRatio)
82: function poolCurrentTick(address pool)
92: function poolsHaveSameTokens(address pool1, address pool2)
```

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/NFTEDA.sol

```
48: function _startAuction(INFTEDA.Auction memory auction)
72: function _purchaseNFT(INFTEDA.Auction memory auction, uint256 maxPrice, address sendTo)
```

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol

```
72: function _setCreatorDiscount(uint256 _auctionCreatorDiscountPercentWad)
```


### G03 MULTIPLICATION or DIVISION BY TWO SHOULD USE BIT SHIFTING

*Number of Instances Identified: 1*

`<x> * 2` is equivalent to `<x> << 1` and `<x> / 2` is the same as `<x> >> 1`. The `MUL` and `DIV` opcodes cost 5 gas, whereas `SHL` and `SHR` only cost 3 gas

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/UniswapHelpers.sol

```
111: return TickMath.getSqrtRatioAtTick(TickMath.getTickAtSqrtRatio(uint160((token1ONE << 96) / token0ONE)) / 2);
```


### G-04 USAGE OF UINTS or INTS SMALLER THAN 32 BYTES - 256 BITS INCURS OVERHEAD

*Number of Instances Identified: 4*

> When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

[https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html](https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html) Use a larger size then downcast where needed

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol

```
83: uint160 initSqrtRatio;
436: uint16 newCount;
```

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol

```
29:  uint128 internal _target;
```

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/UniswapHelpers.sol

```
37: uint160 sqrtPriceLimitX96,
```
