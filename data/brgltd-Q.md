# [01] Lack of check for contract existence for ERC20 transfer

[Solmate](https://github.com/transmissions11/solmate/blob/main/src/utils/SafeTransferLib.sol#L9) won't check for contract existence. If a contract used as underlying ever gets deleted, calling `PaprController.uniswapV3SwapCallback()` will result a silent failure.

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L253

## Recommendation

Add a check for contract existence prior to executing the transfer, e.g. `address(underlying).code.length > 0`.

# [02] Check the return of ecrecover against address(0) 

If the signature for [L68-L86](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/ReservoirOracleUnderwriter.sol#L68-L86) doesn't match, `signerAddress` will be address(0). And if `oracleSigner` is initialized with address(0), the validation on [L88-L90](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/ReservoirOracleUnderwriter.sol#L88-L90) will pass.

## Recommendation

The return of `ecrecoever` should include a address(0) check, e.g.

```
if (signerAddress != address(0) && signerAddress != oracleSigner) {
    revert IncorrectOracleSigner();
}
```

# [03] Lack of checks-effects-interactions

For `PaprController.buyAndReduceDebt()`, the external call `underling.transfer()` is executed before state updates are made into `_reduceDebt()`.

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L225-L229

## Recommendation

Follow the checks-effects-interactions and update state variables prior to external calls.

# [04] Missing event for setter function

Adding an event into `PaprController.setLiquidationLocked()` will facilitate offchain monitoring and indexing.

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L360-L362

# [05] Downcasting block.timestamp to uint48

Downcasting block.timestamp to uint40 or uint48 will not be an issue for a long time. However, the project could use a bigger value (or use the safecast function) to improve the resilience of the protocol - since safecast is not being used, when the time comes, state variables like `_lastUpdate` and `info.lastestAuctionStartTime` will overflow. See this [item](https://code4rena.com/reports/2022-07-juicebox/#l-04-splits-cant-be-locked-once-the-timestamp-passes-typeuint48max) marked as a finding for a previous code4rena QA report.

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L100

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L325

# [06] `safeTransferFrom` can be replace with `safeTransfer`

`papr.safeTransferFrom(address(this), to, amount)` can be replaced with `papr.safeTransfer(to, amount)`

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L383

# [07] Variable shadowing

Consider renaming `underlying` on [L73](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L73). It's being shadowed by `UniswapOracleFundingRateController.underlying`.

# [08] Missing NATSPEC

Consider adding natspec on all external functions to improve documentation.

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L234

# [09] Can use ternary

The following instance

```
uint160 initSqrtRatio;

// initialize the pool at 1:1
if (token0IsUnderlying) {
    initSqrtRatio = UniswapHelpers.oneToOneSqrtRatio(underlyingONE, 10 ** 18);
} else {
    initSqrtRatio = UniswapHelpers.oneToOneSqrtRatio(10 ** 18, underlyingONE);
}
```

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L83-L90

Can be replaced with

```
// initialize the pool at 1:1
uint160 initSqrtRatio = token0IsUnderlying
    ? UniswapHelpers.oneToOneSqrtRatio(underlyingONE, 10 ** 18)
    : UniswapHelpers.oneToOneSqrtRatio(10 ** 18, underlyingONE);
```

# [10] Use custom errors consistently

The protocol is mostly using custom errors. Consider refactoring old revert and require statements to use custom errors consistently throughout the whole codebase.

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L236

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L245

# [11] Use `type(uint200).max`

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L473

# [12] Inconsistent use of named return variables

Some functions return named variables, others return explicit values.

Following function returns an explicit value.

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L208-L212

Following function return returns a named variable.

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L182-L187

Consider adopting the same approach throughout the codebase to improve the explicitness and readability of the code.

# [13] Order of function

The solidity [documentation](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions) recommends the following order for functions:

constructor
receive function (if exists)
fallback function (if exists)
external
public
internal
private

The instances bellow highlight public above external.

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L72

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L81

# [14] Replace magic numbers with constants

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L76

# [15] Move validation statements to the top of the function when possible

Statements used to validate input parameters should be executed before the function internal logic.

For `PaprController.startLiquidationAuction()`, the validation on [L311-L313](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L311-L313) can be move to the top of the function, above [L306-L308](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L306-L308) and bellow [L302-304](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L302-L304).

# [16] Large multiples of ten should use scientific notation.

Using scientific notation for large multiples of ten will improve code readability.

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L92
