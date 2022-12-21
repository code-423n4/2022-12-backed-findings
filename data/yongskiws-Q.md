## Consider Tick Calculations
OracleLibrary.sol

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/OracleLibrary.sol#L10-L31
```  
function getQuoteAtTick(int24 tick, uint128 baseAmount, address baseToken, address quoteToken) internal view returns (uint256) {
    uint256 sqrtRatioX96 = TickMath.getSqrtRatioAtTick(tick);
    uint256 ratio = sqrtRatioX96 * sqrtRatioX96;

    if (baseToken < quoteToken) {
        return FullMath.mulDiv(ratio, baseAmount, 1 << 128);
    } else {
        return FullMath.mulDiv(1 << 128, baseAmount, ratio);
    }
} 
```

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/OracleLibrary.sol#L34-L53

```
function timeWeightedAverageTick(int56 startTick, int56 endTick, int56 twapDuration) internal view returns (int24) {
    require(twapDuration != 0, "BP");

    int56 delta = endTick - startTick;
    return int24(delta / twapDuration);
}
```
