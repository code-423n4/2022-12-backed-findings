
## Summary

### Low Risk Issues
| |Issue|Instances|
|-|:-|:-:|
| [L&#x2011;01] | Return value of `ecrecover()` is not checked | 1 | 

Total: 1 instances over 1 issues


### Non-critical Issues
| |Issue|Instances|
|-|:-|:-:|
| [N&#x2011;01] | `constant`s should be defined rather than using magic numbers | 16 | 
| [N&#x2011;02] | Missing event and or timelock for critical parameter change | 1 | 
| [N&#x2011;03] | Events that mark critical parameter changes should contain both the old and the new value | 2 | 
| [N&#x2011;04] | Use a more recent version of solidity | 3 | 
| [N&#x2011;05] | Use a more recent version of solidity | 1 | 
| [N&#x2011;06] | Use scientific notation (e.g. `1e18`) rather than exponentiation (e.g. `10**18`) | 2 | 
| [N&#x2011;07] | Non-library/interface files should use fixed compiler versions, not floating ones | 4 | 
| [N&#x2011;08] | Using `>`/`>=` without specifying an upper bound is unsafe | 10 | 
| [N&#x2011;09] | Typos | 3 | 
| [N&#x2011;10] | File is missing NatSpec | 3 | 
| [N&#x2011;11] | NatSpec is incomplete | 10 | 
| [N&#x2011;12] | Consider using `delete` rather than assigning zero to clear values | 2 | 
| [N&#x2011;13] | Contracts should have full test coverage | 1 | 
| [N&#x2011;14] | Large or complicated code bases should implement fuzzing tests | 1 | 

Total: 59 instances over 14 issues





## Low Risk Issues

### [L&#x2011;01]  Return value of `ecrecover()` is not checked
When the signature doesn't match, `0` is returned as the signer. Usually this is fine, but in this case, the immutable `oracleSigner` address is not verified to be non-zero, so if a mistake is made during deployment, the end result will be that anyone can specify whatever underwriting price they want, allowing them to manipulate the pool to have whatever value they wish

*There is 1 instance of this issue:*

```solidity
File: /src/ReservoirOracleUnderwriter.sol

88:          if (signerAddress != oracleSigner) {

```
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/ReservoirOracleUnderwriter.sol#L88

## Non-critical Issues

### [N&#x2011;01]  `constant`s should be defined rather than using magic numbers
Even [assembly](https://github.com/code-423n4/2022-05-opensea-seaport/blob/9d7ce4d08bf3c3010304a0476a785c70c0e90ae7/contracts/lib/TokenTransferrer.sol#L35-L39) can benefit from using readable constants instead of hex/numeric literals

*There are 16 instances of this issue:*

```solidity
File: src/libraries/OracleLibrary.sol

/// @audit 192
22:                       ? FullMath.mulDiv(ratioX192, baseAmount, 1 << 192)

/// @audit 192
23:                       : FullMath.mulDiv(1 << 192, baseAmount, ratioX192);

/// @audit 64
25:                   uint256 ratioX128 = FullMath.mulDiv(sqrtRatioX96, sqrtRatioX96, 1 << 64);

/// @audit 128
27:                       ? FullMath.mulDiv(ratioX128, baseAmount, 1 << 128)

/// @audit 128
28:                       : FullMath.mulDiv(1 << 128, baseAmount, ratioX128);

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/libraries/OracleLibrary.sol#L22

```solidity
File: src/libraries/UniswapHelpers.sol

/// @audit 96
111:          return TickMath.getSqrtRatioAtTick(TickMath.getTickAtSqrtRatio(uint160((token1ONE << 96) / token0ONE)) / 2);

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/libraries/UniswapHelpers.sol#L111

```solidity
File: src/PaprController.sol

/// @audit 18
87:               initSqrtRatio = UniswapHelpers.oneToOneSqrtRatio(underlyingONE, 10 ** 18);

/// @audit 18
89:               initSqrtRatio = UniswapHelpers.oneToOneSqrtRatio(10 ** 18, underlyingONE);

/// @audit 10000
92:           address _pool = UniswapHelpers.deployAndInitPool(address(underlying), address(papr), 10000, initSqrtRatio);

/// @audit 1e17
76:           NFTEDAStarterIncentive(1e17)

/// @audit 200
473:          if (newDebt >= 1 << 200) revert IPaprController.DebtAmountExceedsUint200();

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/PaprController.sol#L87

```solidity
File: src/PaprToken.sol

/// @audit 18
19:           ERC20(string.concat("papr ", name), string.concat("papr", symbol), 18)

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/PaprToken.sol#L19

```solidity
File: src/UniswapOracleFundingRateController.sol

/// @audit 4
41:           _setFundingPeriod(4 weeks);

/// @audit 7
123:          if (_fundingPeriod < 7 days) revert FundingPeriodTooShort();

/// @audit 90
124:          if (_fundingPeriod > 90 days) revert FundingPeriodTooLong();

/// @audit 1e18
138:          return OracleLibrary.getQuoteAtTick(twapTick, 1e18, address(papr), address(underlying));

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/UniswapOracleFundingRateController.sol#L41

### [N&#x2011;02]  Missing event and or timelock for critical parameter change
Events help non-contract tools to track changes, and events prevent users from being surprised by changes

*There is 1 instance of this issue:*

```solidity
File: src/PaprController.sol

360       function setLiquidationsLocked(bool locked) external override onlyOwner {
361           liquidationsLocked = locked;
362:      }

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/PaprController.sol#L360-L362

### [N&#x2011;03]  Events that mark critical parameter changes should contain both the old and the new value
This should especially be done if the new value is not required to be different from the old value

*There are 2 instances of this issue:*

```solidity
File: src/PaprController.sol

/// @audit setAllowedCollateral()
374:              emit AllowCollateral(collateralConfigs[i].collateral, collateralConfigs[i].allowed);

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/PaprController.sol#L374

```solidity
File: src/UniswapOracleFundingRateController.sol

/// @audit updateTarget()
59:           emit UpdateTarget(nTarget);

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/UniswapOracleFundingRateController.sol#L59

### [N&#x2011;04]  Use a more recent version of solidity
Use a solidity version of at least 0.8.13 to get the ability to use `using for` with a list of free functions

*There are 3 instances of this issue:*

```solidity
File: src/libraries/UniswapHelpers.sol

2:    pragma solidity >=0.8.0;

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/libraries/UniswapHelpers.sol#L2

```solidity
File: src/NFTEDA/libraries/EDAPrice.sol

2:    pragma solidity >=0.8.0;

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/NFTEDA/libraries/EDAPrice.sol#L2

```solidity
File: src/NFTEDA/NFTEDA.sol

2:    pragma solidity >=0.8.0;

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/NFTEDA/NFTEDA.sol#L2

### [N&#x2011;05]  Use a more recent version of solidity
Use a solidity version of at least 0.8.4 to get `bytes.concat()` instead of `abi.encodePacked(<bytes>,<bytes>)`
Use a solidity version of at least 0.8.12 to get `string.concat()` instead of `abi.encodePacked(<str>,<str>)`

*There is 1 instance of this issue:*

```solidity
File: src/libraries/PoolAddress.sol

4:    pragma solidity >=0.8.0;

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/libraries/PoolAddress.sol#L4

### [N&#x2011;06]  Use scientific notation (e.g. `1e18`) rather than exponentiation (e.g. `10**18`)
While the compiler knows to optimize away the exponentiation, it's still better coding practice to use idioms that do not require compiler optimization, if they exist

*There are 2 instances of this issue:*

```solidity
File: src/PaprController.sol

87:               initSqrtRatio = UniswapHelpers.oneToOneSqrtRatio(underlyingONE, 10 ** 18);

89:               initSqrtRatio = UniswapHelpers.oneToOneSqrtRatio(10 ** 18, underlyingONE);

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/PaprController.sol#L87

### [N&#x2011;07]  Non-library/interface files should use fixed compiler versions, not floating ones

*There are 4 instances of this issue:*

```solidity
File: src/PaprController.sol

2:    pragma solidity ^0.8.17;

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/PaprController.sol#L2

```solidity
File: src/PaprToken.sol

2:    pragma solidity ^0.8.17;

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/PaprToken.sol#L2

```solidity
File: src/ReservoirOracleUnderwriter.sol

2:    pragma solidity ^0.8.17;

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/ReservoirOracleUnderwriter.sol#L2

```solidity
File: src/UniswapOracleFundingRateController.sol

2:    pragma solidity ^0.8.17;

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/UniswapOracleFundingRateController.sol#L2

### [N&#x2011;08]  Using `>`/`>=` without specifying an upper bound is unsafe
There _will_ be breaking changes in future versions of solidity, and at that point your code will no longer be compatable. While you may have the specific version to use in a configuration file, others that include your source files may not.

*There are 10 instances of this issue:*

```solidity
File: src/interfaces/IFundingRateController.sol

2:    pragma solidity >=0.8.0;

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/interfaces/IFundingRateController.sol#L2

```solidity
File: src/interfaces/IPaprController.sol

2:    pragma solidity >=0.8.0;

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/interfaces/IPaprController.sol#L2

```solidity
File: src/interfaces/IUniswapOracleFundingRateController.sol

2:    pragma solidity >=0.8.0;

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/interfaces/IUniswapOracleFundingRateController.sol#L2

```solidity
File: src/libraries/OracleLibrary.sol

2:    pragma solidity >=0.8.0;

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/libraries/OracleLibrary.sol#L2

```solidity
File: src/libraries/PoolAddress.sol

4:    pragma solidity >=0.8.0;

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/libraries/PoolAddress.sol#L4

```solidity
File: src/libraries/UniswapHelpers.sol

2:    pragma solidity >=0.8.0;

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/libraries/UniswapHelpers.sol#L2

```solidity
File: src/NFTEDA/extensions/NFTEDAStarterIncentive.sol

2:    pragma solidity >=0.8.0;

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol#L2

```solidity
File: src/NFTEDA/interfaces/INFTEDA.sol

2:    pragma solidity >=0.8.0;

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/NFTEDA/interfaces/INFTEDA.sol#L2

```solidity
File: src/NFTEDA/libraries/EDAPrice.sol

2:    pragma solidity >=0.8.0;

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/NFTEDA/libraries/EDAPrice.sol#L2

```solidity
File: src/NFTEDA/NFTEDA.sol

2:    pragma solidity >=0.8.0;

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/NFTEDA/NFTEDA.sol#L2

### [N&#x2011;09]  Typos

*There are 3 instances of this issue:*

```solidity
File: src/NFTEDA/interfaces/INFTEDA.sol

/// @audit Identitical
58:       /// @dev Derived from the auction. Identitical auctions cannot exist simultaneously

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/NFTEDA/interfaces/INFTEDA.sol#L58

```solidity
File: src/NFTEDA/NFTEDA.sol

/// @audit defintion
46:       /// @param auction The defintion of the auction

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/NFTEDA/NFTEDA.sol#L46

```solidity
File: src/PaprController.sol

/// @audit succesful
158:      /// @return selector indicating succesful receiving of the NFT

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/PaprController.sol#L158

### [N&#x2011;10]  File is missing NatSpec

*There are 3 instances of this issue:*

```solidity
File: src/libraries/OracleLibrary.sol


```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/libraries/OracleLibrary.sol

```solidity
File: src/NFTEDA/libraries/EDAPrice.sol


```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/NFTEDA/libraries/EDAPrice.sol

```solidity
File: src/PaprToken.sol


```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/PaprToken.sol

### [N&#x2011;11]  NatSpec is incomplete

*There are 10 instances of this issue:*

```solidity
File: src/interfaces/IPaprController.sol

/// @audit Missing: '@param oracleInfo'
173       /// @notice purchases a liquidation auction with the controller's papr token
174       /// @dev oracleInfo price must be type TWAP
175       /// @param auction auction to purchase
176       /// @param maxPrice maximum price to pay for the auction
177       /// @param sendTo address to send the collateral to if auction is won
178       function purchaseLiquidationAuctionNFT(
179           INFTEDA.Auction calldata auction,
180           uint256 maxPrice,
181           address sendTo,
182:          ReservoirOracleUnderwriter.OracleInfo calldata oracleInfo

/// @audit Missing: '@return'
229       /// @param collateral address of the collateral
230       /// @param tokenId tokenId of the collateral
231:      function collateralOwner(ERC721 collateral, uint256 tokenId) external view returns (address);

/// @audit Missing: '@return'
233       /// @notice returns whether a token address is allowed to serve as collateral for a vault
234       /// @param collateral address of the collateral token
235:      function isAllowed(address collateral) external view returns (bool);

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/interfaces/IPaprController.sol#L173-L182

```solidity
File: src/NFTEDA/interfaces/INFTEDA.sol

/// @audit Missing: '@return'
63        /// @notice Returns the time at which startAuction was most recently successfully called for the given auction id
64        /// @param id The id of the auction
65:       function auctionStartTime(uint256 id) external view returns (uint256);

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/NFTEDA/interfaces/INFTEDA.sol#L63-L65

```solidity
File: src/NFTEDA/NFTEDA.sol

/// @audit Missing: '@param sendTo'
/// @audit Missing: '@return'
69        /// @notice purchases the NFT being sold in `auction`, reverts if current auction price exceed maxPrice
70        /// @param auction The auction selling the NFT
71        /// @param maxPrice The maximum the caller is willing to pay
72        function _purchaseNFT(INFTEDA.Auction memory auction, uint256 maxPrice, address sendTo)
73            internal
74            virtual
75:           returns (uint256 startTime, uint256 price)

/// @audit Missing: '@param id'
108       /// @notice Returns the current price of the passed auction, reverts if no such auction exists
109       /// @dev startTime is passed, optimized for cases where the auctionId has already been computed
110       /// @dev and startTime looked it up
111       /// @param startTime The start time of the auction
112       /// @param auction The auction for which the caller wants to know the current price
113       /// @return price the current amount required to purchase the NFT being sold in this auction
114       function _auctionCurrentPrice(uint256 id, uint256 startTime, INFTEDA.Auction memory auction)
115           internal
116           view
117           virtual
118:          returns (uint256)

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/NFTEDA/NFTEDA.sol#L69-L75

```solidity
File: src/PaprController.sol

/// @audit Missing: '@param address'
152       /// @notice Handler for safeTransferFrom of an NFT
153       /// @dev Should be preferred to `addCollateral` if only one NFT is being added
154       /// to avoid approval call and save gas
155       /// @param from the current owner of the nft
156       /// @param _id the id of the NFT
157       /// @param data encoded IPaprController.OnERC721ReceivedArgs
158       /// @return selector indicating succesful receiving of the NFT
159       function onERC721Received(address from, address, uint256 _id, bytes calldata data)
160           external
161           override
162:          returns (bytes4)

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/PaprController.sol#L152-L162

```solidity
File: src/UniswapOracleFundingRateController.sol

/// @audit Missing: '@param _mark_'
/// @audit Missing: '@param cachedTarget'
149       /// @notice The multiplier to apply to target() to get newTarget()
150       /// @dev Computes the funding rate for the time since _lastUpdates
151       /// 1 = 1e18, i.e.
152       /// > 1e18 means positive funding rate
153       /// < 1e18 means negative funding rate
154       /// sub 1e18 to get percent change
155       /// @return multiplier used to obtain newTarget()
156:      function _multiplier(uint256 _mark_, uint256 cachedTarget) internal view returns (uint256) {

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/UniswapOracleFundingRateController.sol#L149-L156

### [N&#x2011;12]  Consider using `delete` rather than assigning zero to clear values
The `delete` keyword more closely matches the semantics of what is being done, and draws more attention to the changing of state, which may lead to a more thorough audit of its associated logic

*There are 2 instances of this issue:*

```solidity
File: src/libraries/OracleLibrary.sol

58:           secondAgos[0] = 0;

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/libraries/OracleLibrary.sol#L58

```solidity
File: src/PaprController.sol

526:              _vaultInfo[auction.nftOwner][auction.auctionAssetContract].latestAuctionStartTime = 0;

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/PaprController.sol#L526

### [N&#x2011;13]  Contracts should have full test coverage
While 100% code coverage does not guarantee that there are no bugs, it often will catch easy-to-find bugs, and will ensure that there are fewer regressions when the code invariably has to be modified. Furthermore, in order to get full coverage, code authors will often have to re-organize their code so that it is more modular, so that each component can be tested separately, which reduces interdependencies between modules and layers, and makes for code that is easier to reason about and audit.

*There is 1 instance of this issue:*

```solidity
File: Various Files


```

### [N&#x2011;14]  Large or complicated code bases should implement fuzzing tests
Large code bases, or code with lots of inline-assembly, complicated math, or complicated interactions between multiple contracts, should implement [fuzzing tests](https://medium.com/coinmonks/smart-contract-fuzzing-d9b88e0b0a05). Fuzzers such as Echidna require the test writer to come up with invariants which should not be violated under any circumstances, and the fuzzer tests various inputs and function calls to ensure that the invariants always hold. Even code with 100% code coverage can still have bugs due to the order of the operations a user performs, and fuzzers, with properly and extensively-written invariants, can close this testing gap significantly.

*There is 1 instance of this issue:*

```solidity
File: Various Files


```


___

## Excluded findings
These findings are excluded from awards calculations because there are publicly-available automated tools that find them. The valid ones appear here for completeness

## Summary

### Low Risk Issues
| |Issue|Instances|
|-|:-|:-:|
| [L&#x2011;01] | Missing checks for `address(0x0)` when assigning values to `address` state variables | 3 | 

Total: 3 instances over 1 issues


### Non-critical Issues
| |Issue|Instances|
|-|:-|:-:|
| [N&#x2011;01] | `require()`/`revert()` statements should have descriptive reason strings | 2 | 
| [N&#x2011;02] | `public` functions not called by the contract should be declared `external` instead | 1 | 
| [N&#x2011;03] | Event is missing `indexed` fields | 7 | 

Total: 10 instances over 3 issues





## Low Risk Issues

### [L&#x2011;01]  Missing checks for `address(0x0)` when assigning values to `address` state variables

*There are 3 instances of this issue:*

```solidity
File: src/ReservoirOracleUnderwriter.sol

/// @audit (valid but excluded finding)
52:           oracleSigner = _oracleSigner;

/// @audit (valid but excluded finding)
53:           quoteCurrency = _quoteCurrency;

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/ReservoirOracleUnderwriter.sol#L52

```solidity
File: src/UniswapOracleFundingRateController.sol

/// @audit (valid but excluded finding)
115:          pool = _pool;

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/UniswapOracleFundingRateController.sol#L115

## Non-critical Issues

### [N&#x2011;01]  `require()`/`revert()` statements should have descriptive reason strings

*There are 2 instances of this issue:*

```solidity
File: src/libraries/PoolAddress.sol

/// @audit (valid but excluded finding)
32:           require(key.token0 < key.token1);

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/libraries/PoolAddress.sol#L32

```solidity
File: src/PaprController.sol

/// @audit (valid but excluded finding)
245:              require(amount1Delta > 0); // swaps entirely within 0-liquidity regions are not supported

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/PaprController.sol#L245

### [N&#x2011;02]  `public` functions not called by the contract should be declared `external` instead
Contracts [are allowed](https://docs.soliditylang.org/en/latest/contracts.html#function-overriding) to override their parents' functions and change the visibility from `external` to `public`.

*There is 1 instance of this issue:*

```solidity
File: src/UniswapOracleFundingRateController.sol

/// @audit (valid but excluded finding)
72:       function mark() public view returns (uint256) {

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/UniswapOracleFundingRateController.sol#L72

### [N&#x2011;03]  Event is missing `indexed` fields
Index event fields make the field more quickly accessible [to off-chain tools](https://ethereum.stackexchange.com/questions/40396/can-somebody-please-explain-the-concept-of-event-indexing) that parse events. However, note that each index field costs extra gas during emission, so it's not necessarily best to index the maximum allowed per event (three fields). Each `event` should use three `indexed` fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three fields, all of the fields should be indexed.

*There are 7 instances of this issue:*

```solidity
File: src/interfaces/IFundingRateController.sol

/// @audit (valid but excluded finding)
9:        event UpdateTarget(uint256 newTarget);

/// @audit (valid but excluded finding)
11:       event SetFundingPeriod(uint256 fundingPeriod);

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/interfaces/IFundingRateController.sol#L9

```solidity
File: src/interfaces/IPaprController.sol

/// @audit (valid but excluded finding)
67:       event IncreaseDebt(address indexed account, ERC721 indexed collateralAddress, uint256 amount);

/// @audit (valid but excluded finding)
85:       event ReduceDebt(address indexed account, ERC721 indexed collateralAddress, uint256 amount);

/// @audit (valid but excluded finding)
90:       event AllowCollateral(address indexed collateral, bool isAllowed);

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/interfaces/IPaprController.sol#L67

```solidity
File: src/NFTEDA/extensions/NFTEDAStarterIncentive.sol

/// @audit (valid but excluded finding)
21:       event SetAuctionCreatorDiscount(uint256 discount);

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol#L21

```solidity
File: src/NFTEDA/interfaces/INFTEDA.sol

/// @audit (valid but excluded finding)
50:       event EndAuction(uint256 indexed auctionID, uint256 price);

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/NFTEDA/interfaces/INFTEDA.sol#L50
