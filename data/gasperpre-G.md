## Use `_pool` parameter instead of `pool` storage variable
In `UniswapOracleFundingRateController`, the `init` function takes `_pool` address as parameter, and saves it to state variable `pool`.  It then proceeds to read `pool` storage variable two times, when it could use `_pool` parameter instead.

### Recommendation
Change the lines 102 and 103 like so:
```
_lastCumulativeTick = OracleLibrary.latestCumulativeTick(_pool);
_lastTwapTick = UniswapHelpers.poolCurrentTick(_pool);
```