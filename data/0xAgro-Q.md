# QA Report
## Finding Summary
||Issue|Instances|
|-|-|-|
|[NC-01]|Long Lines (> 120 Characters)|7|
|[NC-02]|Spelling Mistakes|3|
|[NC-03]|Trailing `.` In NatSpec Voids General Style|2|
|[NC-04]|Power of Ten Literal > `10e3` Not In Scientific Notation|1|
|[NC-05]|Order of Functions Not Compliant With Solidity Docs|1|

### [NC-01] Long Lines (> 120 Characters)

Lines with greater length than 120 characters are used. The [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-lengthhttps://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-length) suggests that all lines should be 120 characters or less in width.

#### Findings:

*/src/PaprController.sol*
Links: [77](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L77), [122](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L122).
```solidity
77:	        UniswapOracleFundingRateController(underlying, new PaprToken(name, symbol), indexMarkRatioMax, indexMarkRatioMin)
122:	                    underwritePriceForCollateral(collateralAddr, ReservoirOracleUnderwriter.PriceKind.LOWER, oracleInfo);
```

*/src/interfaces/IPaprController.sol*
Links: [66](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/interfaces/IPaprController.sol#L66), [164](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/interfaces/IPaprController.sol#L164), [251](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/interfaces/IPaprController.sol#L251), [257](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/interfaces/IPaprController.sol#L257), [260](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/interfaces/IPaprController.sol#L260).
```solidity
66:	    /// @dev vaults are uniquely identified by the address of the vault owner and the address of the collateral token used in the vault
164:	    /// @notice removes debt from a vault and burns it by buying it on Uniswap in exchange for the controller's underlying token
251:	    /// @notice amount the price of an auction decreases by per auctionDecayPeriod, expressed as a decimal scaled by 1e18
257:	    /// @notice the multiplier for the starting price of an auction, applied to the current price of the collateral in papr tokens
260:	    /// @notice fee paid by the vault owner when their vault is liquidated if there was excess debt credited to their vault, in bips
```

### [NC-02] Spelling Mistakes

There are a few spelling mistakes throughout the codebase. Consider fixing all spelling mistakes.

#### Findings:

The word `successful` is misspelled as `succesful`.

*/src/PaprController.sol*
Links: [158](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L158).
```solidity
158:	/// @return selector indicating succesful receiving of the NFT
```

The word `definition` is misspelled as `defintion`.

*/src/NFTEDA/NFTEDA.sol*
Links: [46](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/NFTEDA.sol#L46).
```solidity
158:	/// @param auction The defintion of the auction
```

The word `Identical` is misspelled as `Identitical`.

*/src/NFTEDA/interfaces/INFTEDA.sol*
Links: [58](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/interfaces/INFTEDA.sol#L58).
```solidity
58:	/// @dev Derived from the auction. Identitical auctions cannot exist simultaneously
```

### [NC-03] Trailing `.` In NatSpec Voids General Style

There are times where NatSpec comments end with a `.` in the codebase; however, most do not. Consider removing any `.` that swims away from the general style.

#### Findings

*/src/NFTEDA/NFTEDA.sol*
Links: [44](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/NFTEDA.sol#L44).
```solidity
44:	/// @dev does no validation the auction, aside that it does not exist.
```

*/src/interfaces/IFundingRateController.sol*
Links; [28](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/interfaces/IFundingRateController.sol#L28), [29](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/interfaces/IFundingRateController.sol#L29).
```solidity
28:	/// value, then funding rates are 0 and newTarget() will equal target().
29:	/// @return target The value of one whole unit of papr in underlying units.
```

### [NC-04] Power of Ten Literal > `10e3` Not In Scientific Notation

Power of ten literals > `10e3` are easier to read when expressed in scientific notation. Consider expressing large powers of ten in scientific notation (Ex. 10e5).

#### Findings:

*/src/PaprController.sol*
Links: [92](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L92).
```solidity
92:	address _pool = UniswapHelpers.deployAndInitPool(address(underlying), address(papr), 10000, initSqrtRatio);
```

### [NC-05] Order of Functions Not Compliant With Solidity Docs

The [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions) suggests the following function ordering:  constructor, receive function (if exists), fallback function (if exists), external, public, internal, private.

#### Findings:

The following contracts are not compliant (examples are only to prove the functions are out of order NOT a full description): 

[UniswapOracleFundingRateController.sol](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol): external functions are positioned after public functions.