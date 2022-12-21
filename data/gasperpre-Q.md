## Variable shadowing
There are two cases of variable shadowing throughout the codebase.

In `UniswapOracleFundingRateController.sol` [L95 `uint256 target`](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L95) shadows [L86 `function target`](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L86)

In `PaprController.sol` [L73 `ERC20 underlying`](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L73) shadows [L17 `ERC20 public immutable underlying` inherited from `UniswapOracleFundingRateController.sol` ](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L17)

### Recommendation
Avoid shadowing by adding prefix `_` for function parameters.