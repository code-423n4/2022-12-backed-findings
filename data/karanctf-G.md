## `10eX` use less gas then `10**X`
**summary**:Using the `**` operator in Solidity can be more expensive in terms of gas consumption than using the `e` notation to represent the same value, because the `**` operator requires additional computation by the EVM.

```solidity
src/PaprController.sol:82:        uint256 underlyingONE = 10 ** underlying.decimals();
src/PaprController.sol:87:            initSqrtRatio = UniswapHelpers.oneToOneSqrtRatio(underlyingONE, 10 ** 18);
src/PaprController.sol:89:            initSqrtRatio = UniswapHelpers.oneToOneSqrtRatio(10 ** 18, underlyingONE);
```


