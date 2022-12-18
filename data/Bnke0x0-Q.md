

### [L01] Missing checks for `address(0x0)` when assigning values to `address` state variables


#### Findings:
```
papr/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol::73 => auctionCreatorDiscountPercentWad = _auctionCreatorDiscountPercentWad;
papr/src/PaprController.sol::80 => maxLTV = _maxLTV;
papr/src/ReservoirOracleUnderwriter.sol::52 => oracleSigner = _oracleSigner;
papr/src/ReservoirOracleUnderwriter.sol::53 => quoteCurrency = _quoteCurrency;
papr/src/UniswapOracleFundingRateController.sol::35 => underlying = _underlying;
papr/src/UniswapOracleFundingRateController.sol::36 => papr = _papr;
papr/src/UniswapOracleFundingRateController.sol::38 => targetMarkRatioMax = _targetMarkRatioMax;
papr/src/UniswapOracleFundingRateController.sol::39 => targetMarkRatioMin = _targetMarkRatioMin;
papr/src/UniswapOracleFundingRateController.sol::115 => pool = _pool;
papr/src/UniswapOracleFundingRateController.sol::126 => fundingPeriod = _fundingPeriod;
```





### [L02] `_safeMint()` should be used rather than `_mint()` wherever possible

#### Impact
_mint() is discouraged in favor of _safeMint() which ensures that the recipient is either an EOA or implements IERC721Receiver. 
#### Findings:
```
papr/src/PaprToken.sol::25 => _mint(to, amount);
```


### 'Non-Critical Issues'

### [N01] Adding a return statement when the function defines a named return variable, is redundant


#### Findings:
```
papr/src/PaprController.sol::231 => return amountOut;
papr/src/PaprController.sol::529 => return price;
papr/src/PaprController.sol::534 => returns (uint256 remaining)
papr/src/ReservoirOracleUnderwriter.sol::116 => return oraclePrice;
papr/src/UniswapOracleFundingRateController.sol::47 => return _target;
papr/src/UniswapOracleFundingRateController.sol::65 => return _target;
papr/src/UniswapOracleFundingRateController.sol::82 => return _lastUpdated;
papr/src/UniswapOracleFundingRateController.sol::87 => return _target;
```



### [N02] `require()`/`revert()` statements should have descriptive reason strings


#### Findings:
```
papr/src/PaprController.sol::245 => require(amount1Delta > 0); // swaps entirely within 0-liquidity regions are not supported
papr/src/libraries/PoolAddress.sol::32 => require(key.token0 < key.token1);
```



### [N03] constants should be defined rather than using magic numbers


#### Findings:
```

papr/src/PaprController.sol::44 => uint256 public immutable override perPeriodAuctionDecayWAD = 0.7e18;
papr/src/PaprController.sol::82 => uint256 underlyingONE = 10 ** underlying.decimals();
papr/src/PaprController.sol::87 => initSqrtRatio = UniswapHelpers.oneToOneSqrtRatio(underlyingONE, 10 ** 18);
papr/src/PaprController.sol::89 => initSqrtRatio = UniswapHelpers.oneToOneSqrtRatio(10 ** 18, underlyingONE);
papr/src/PaprController.sol::92 => address _pool = UniswapHelpers.deployAndInitPool(address(underlying), address(papr), 10000, initSqrtRatio);
papr/src/PaprToken.sol::19 => ERC20(string.concat("papr ", name), string.concat("papr", symbol), 18)
papr/src/UniswapOracleFundingRateController.sol::138 => return OracleLibrary.getQuoteAtTick(twapTick, 1e18, address(papr), address(underlying));
papr/src/libraries/OracleLibrary.sol::25 => uint256 ratioX128 = FullMath.mulDiv(sqrtRatioX96, sqrtRatioX96, 1 << 64);
papr/src/libraries/OracleLibrary.sol::27 => ? FullMath.mulDiv(ratioX128, baseAmount, 1 << 128)
papr/src/libraries/OracleLibrary.sol::28 => : FullMath.mulDiv(1 << 128, baseAmount, ratioX128);
```






### [N04] Variable names that consist of all capital letters should be reserved for `const`/`immutable `variables

#### Impact
If the variable needs to be different based on which class it comes from, a view/pure function should be used instead (e.g. like [this](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/76eee35971c2541585e05cbf258510dda7b2fbc6/contracts/token/ERC20/extensions/draft-IERC20Permit.sol#L59)).
#### Findings:
```
papr/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol::27 => uint256 internal _pricePercentAfterDiscount;
papr/src/UniswapOracleFundingRateController.sol::29 => uint128 internal _target;
papr/src/UniswapOracleFundingRateController.sol::30 => int56 internal _lastCumulativeTick;
papr/src/UniswapOracleFundingRateController.sol::31 => uint48 internal _lastUpdated;
papr/src/UniswapOracleFundingRateController.sol::32 => int24 internal _lastTwapTick;
```



### [N05] File is missing NatSpec


#### Findings:
```
papr/src/libraries/OracleLibrary.sol::1 => // SPDX-License-Identifier: GPL-2.0-or-later
papr/src/libraries/PoolAddress.sol::1 => // SPDX-License-Identifier: GPL-2.0-or-later
papr/src/libraries/UniswapHelpers.sol::1 => // SPDX-License-Identifier: GPL-2.0-or-later
```



### [N06] Unused file


#### Findings:
```
papr/src/NFTEDA/NFTEDA.sol::1 => // SPDX-License-Identifier: MIT
papr/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol::1 => // SPDX-License-Identifier: MIT
papr/src/NFTEDA/interfaces/INFTEDA.sol::1 => // SPDX-License-Identifier: MIT
papr/src/NFTEDA/libraries/EDAPrice.sol::1 => // SPDX-License-Identifier: MIT
papr/src/PaprToken.sol::1 => // SPDX-License-Identifier: MIT
papr/src/ReservoirOracleUnderwriter.sol::1 => // SPDX-License-Identifier: MIT
papr/src/interfaces/IFundingRateController.sol::1 => // SPDX-License-Identifier: MIT
papr/src/interfaces/IPaprController.sol::1 => // SPDX-License-Identifier: MIT
papr/src/interfaces/IUniswapOracleFundingRateController.sol::1 => // SPDX-License-Identifier: MIT
```



### [N07] `public` functions not called by the contract should be declared `external` instead

#### Impact
ontracts are allowed to override their parents’ functions and change the visibility from public to external .
#### Findings:
```
papr/src/NFTEDA/NFTEDA.sol::24 => function auctionCurrentPrice(INFTEDA.Auction calldata auction) public view virtual returns (uint256) {
papr/src/NFTEDA/NFTEDA.sol::35 => function auctionID(INFTEDA.Auction memory auction) public pure virtual returns (uint256) {
papr/src/NFTEDA/NFTEDA.sol::40 => function auctionStartTime(uint256 id) public view virtual returns (uint256);
papr/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol::38 => function auctionStartTime(uint256 id) public view virtual override returns (uint256) {
papr/src/UniswapOracleFundingRateController.sol::45 => function updateTarget() public override returns (uint256 nTarget) {
papr/src/UniswapOracleFundingRateController.sol::63 => function newTarget() public view override returns (uint256) {
papr/src/UniswapOracleFundingRateController.sol::72 => function mark() public view returns (uint256) {
```



### [N08] Numeric values having to do with time should use time units for readability

#### Impact
There are units for seconds, minutes, hours, days, and weeks
#### Findings:
```
papr/src/NFTEDA/interfaces/INFTEDA.sol::20 => // the number of seconds in the period over which perPeriodDecay occurs
papr/src/NFTEDA/interfaces/INFTEDA.sol::34 => /// @param secondsInPeriod the number of seconds in the period over which perPeriodDecay occurs
papr/src/PaprController.sol::41 => uint256 public immutable override liquidationAuctionMinSpacing = 2 days;
papr/src/PaprController.sol::47 => uint256 public immutable override auctionDecayPeriod = 1 days;
papr/src/ReservoirOracleUnderwriter.sol::35 => uint256 constant TWAP_SECONDS = 30 days;
papr/src/ReservoirOracleUnderwriter.sol::38 => uint256 constant VALID_FOR = 20 minutes;
papr/src/UniswapOracleFundingRateController.sol::41 => _setFundingPeriod(4 weeks);
papr/src/UniswapOracleFundingRateController.sol::121 => /// @dev reverts if period is longer than 90 days or less than 7
papr/src/UniswapOracleFundingRateController.sol::123 => if (_fundingPeriod < 7 days) revert FundingPeriodTooShort();
papr/src/UniswapOracleFundingRateController.sol::124 => if (_fundingPeriod > 90 days) revert FundingPeriodTooLong();
papr/src/interfaces/IFundingRateController.sol::23 => /// @return lastUpdated the timestamp (in seconds) at which target was last updated
papr/src/interfaces/IFundingRateController.sol::56 => /// @return fundingPeriod in seconds over which funding is paid
papr/src/interfaces/IPaprController.sol::203 => /// @param _fundingPeriod new funding period in seconds
papr/src/interfaces/IPaprController.sol::254 => /// @notice amount of time that perPeriodAuctionDecayWAD is applied to, expressed in seconds
papr/src/libraries/PoolAddress.sol::19 => /// @param tokenB The second token of a pool, unsorted
papr/src/libraries/UniswapHelpers.sol::65 => /// @param tokenB the second token in the pool
papr/src/libraries/UniswapHelpers.sol::90 => /// @param pool2 the second pool
```






