## Inadequate NatSpec
Solidity contracts can use a special form of comments, i.e., the Ethereum Natural Language Specification Format (NatSpec) to provide rich documentation for functions, return variables and more. Please visit the following link for further details:

https://docs.soliditylang.org/en/v0.8.16/natspec-format.html

Here is a contract instance with missing NatSpec in its entirety:

[File: PaprToken.sol](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprToken.sol)

Here is a contract instance partially missing NatSpec:

[File: PaprController.sol](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol)

```
234:    function uniswapV3SwapCallback(int256 amount0Delta, int256 amount1Delta, bytes calldata _data) external {

All internal view and non-view functions
```
## Unspecific compiler version pragma
The compiler version pragma of [Version.sol](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/utils/Version.sol) is unspecific ^0.8.0 or ^0.8.17. While this often makes sense for libraries to allow them to be included with multiple different versions of an application, it may be a security risk for the actual application implementation itself. A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up actually checking a different EVM compilation that is ultimately deployed on the blockchain.

Avoid floating pragmas where possible. It is highly recommend pinning a concrete compiler version (latest without security issues) in at least the top-level “deployed” contracts to make it unambiguous which compiler version is being used. Rule of thumb: a flattened source-unit should have at least one non-floating concrete solidity compiler version pragma.

## `underwritePriceForCollateral()` does not check if ecrecover return value is 0
In `ReservoirOracleUnderwriter.sol`, `underwritePriceForCollateral()` calls the Solidity ecrecover function directly to verify the given signatures.
The return value of ecrecover may be 0, which means the signature is invalid, but the check can be bypassed when signer is 0.

[File: ReservoirOracleUnderwriter.sol#L67-L90](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/ReservoirOracleUnderwriter.sol#L67-L90)

```
  {
        address signerAddress = ecrecover(
            keccak256(
                abi.encodePacked(
                    "\x19Ethereum Signed Message:\n32",
                    // EIP-712 structured-data hash
                    keccak256(
                        abi.encode(
                            keccak256("Message(bytes32 id,bytes payload,uint256 timestamp)"),
                            oracleInfo.message.id,
                            keccak256(oracleInfo.message.payload),
                            oracleInfo.message.timestamp
                        )
                    )
                )
            ),
            oracleInfo.sig.v,
            oracleInfo.sig.r,
            oracleInfo.sig.s
        );

        if (signerAddress != oracleSigner) {
            revert IncorrectOracleSigner();
        }
```
Consider using the recover function from OpenZeppelin's ECDSA library for signature verification.

Alternatively, the affected code line may be refactored as follows:

```diff
-        if (signerAddress != oracleSigner) {
+        if (signerAddress != oracleSigner || signerAddress == address(0)) {
            revert IncorrectOracleSigner();
        }
```
## Events associated with setter functions
Consider having events associated with setter functions emit both the new and old values instead of just the new value.

Here are some of the instances entailed:

[File: UniswapOracleFundingRateController.sol](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol)

```
117:        emit SetPool(_pool);

128:        emit SetFundingPeriod(_fundingPeriod);
```
## Sanity checks at the constructor
Adequate zero address and zero value checks should be implemented at the constructor to avoid accidental error(s) that could result in non-functional calls associated with it particularly when assigning immutable variables.

Here are some of the instances entailed:

[File: UniswapOracleFundingRateController.sol#L34-L42](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L34-L42)

```
    constructor(ERC20 _underlying, ERC20 _papr, uint256 _targetMarkRatioMax, uint256 _targetMarkRatioMin) {
        underlying = _underlying;
        papr = _papr;

        targetMarkRatioMax = _targetMarkRatioMax;
        targetMarkRatioMin = _targetMarkRatioMin;

        _setFundingPeriod(4 weeks);
    }
```
## Immutable variables should be parameterized in the constructor
Consider assigning immutable variables in the constructor via parameter inputs instead of directly assigning them with literal values. If the latter approach is preferred, consider declaring them as constants.

Here are some of the instances entailed:

[File: PaprController.sol#L41-L54](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L41-L54)

```
    uint256 public immutable override liquidationAuctionMinSpacing = 2 days;

    /// @inheritdoc IPaprController
    uint256 public immutable override perPeriodAuctionDecayWAD = 0.7e18;

    /// @inheritdoc IPaprController
    uint256 public immutable override auctionDecayPeriod = 1 days;

    /// @inheritdoc IPaprController
    uint256 public immutable override auctionStartPriceMultiplier = 3;

    /// @inheritdoc IPaprController
    /// @dev Set to 10%
    uint256 public immutable override liquidationPenaltyBips = 1000;
```
## Constants should be defined rather than using magic numbers
Consider defining magic numbers to constants, serving to improve code readability and maintainability.

Here are some of the instances entailed:

[File: PaprController.sol](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol)

```
87:            initSqrtRatio = UniswapHelpers.oneToOneSqrtRatio(underlyingONE, 10 ** 18);

89:            initSqrtRatio = UniswapHelpers.oneToOneSqrtRatio(10 ** 18, underlyingONE);

92:        address _pool = UniswapHelpers.deployAndInitPool(address(underlying), address(papr), 10000, initSqrtRatio);
```
## Use `delete` to clear variables
`delete a` assigns the initial value for the type to `a`. i.e. for integers it is equivalent to `a = 0`, but it can also be used on arrays, where it assigns a dynamic array of length zero or a static array of the same length with all elements reset. For structs, it assigns a struct with all members reset. Similarly, it can also be used to set an address to zero address or a boolean to false. It has no effect on whole mappings though (as the keys of mappings may be arbitrary and are generally unknown). However, individual keys and what they map to can be deleted: If `a` is a mapping, then `delete a[x]` will delete the value stored at x.

The delete key better conveys the intention and is also more idiomatic.

For instance, the `a = 0` instance below may be refactored as follows:

[File: PaprController.sol#L526](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L526)

```diff
-            _vaultInfo[auction.nftOwner][auction.auctionAssetContract].latestAuctionStartTime = 0;
+            delete _vaultInfo[auction.nftOwner][auction.auctionAssetContract].latestAuctionStartTime ;
```
## Solmate's contract existence check
As denoted by [File: SafeTransferLib.sol#L9](https://github.com/transmissions11/solmate/blob/main/src/utils/SafeTransferLib.sol#L9):

```
/// @dev Note that none of the functions in this library check that a token has code at all! That responsibility is delegated to the caller.
```
As such, the following token transfers associated with Solmate's `SafeTransferLib.sol`, are performing [low-level calls without confirming contract’s existence that could return success even though no function call was executed](https://docs.soliditylang.org/en/v0.8.7/control-structures.html#error-handling-assert-require-revert-and-exceptions):

[File: PaprController.sol](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol)

```
202:            underlying.transfer(params.swapFeeTo, fee);
203:            underlying.transfer(proceedsTo, amountOut - fee);

226:            underlying.transfer(params.swapFeeTo, amountIn * params.swapFeeBips / BIPS_ONE);

253:            underlying.safeTransferFrom(payer, msg.sender, amountToPay);

383:        papr.safeTransferFrom(address(this), to, amount);

514:            underlying.transfer(params.swapFeeTo, fee);
515:            underlying.transfer(proceedsTo, amountOut - fee);

546:            papr.transfer(auction.nftOwner, totalOwed - debtCached);
```
## Minimization of truncation
The number of divisions in an equation should be reduced to minimize truncation frequency, serving to achieve higher precision. And, where deemed fit, comment the code line with the original multiple division arithmetic operation for clarity reason.

For instance, the code line below with two division may be refactored as follows:

[File: UniswapHelpers.sol#L111](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/UniswapHelpers.sol#L111)

```diff
-        return TickMath.getSqrtRatioAtTick(TickMath.getTickAtSqrtRatio(uint160((token1ONE << 96) / token0ONE)) / 2);
+        return TickMath.getSqrtRatioAtTick(TickMath.getTickAtSqrtRatio(uint160((token1ONE << 96) / (token0ONE * 2));
```