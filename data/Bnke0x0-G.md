

### [G01] State variables only set in the constructor should be declared `immutable`

#### Impact
Avoids a Gusset (20000 gas)
#### Findings:
```
papr/src/PaprController.sol::32 => bool public override liquidationsLocked;
papr/src/UniswapOracleFundingRateController.sol::21 => uint256 public fundingPeriod;
papr/src/UniswapOracleFundingRateController.sol::23 => address public pool;
```



### [G02] State variables can be packed into fewer storage slots

#### Impact
If variables occupying the same slot are both written the same 
function or by the constructor avoids a separate Gsset (20000 gas). 
Reads of the variables are also cheaper
#### Findings:
```
papr/src/UniswapOracleFundingRateController.sol::23 => address public pool;
```





### [G03] `<array>.length` should not be looked up in every loop of a `for` loop


#### Findings:
```
papr/src/PaprController.sol::99 => for (uint256 i = 0; i < collateralArr.length;) {
papr/src/PaprController.sol::118 => for (uint256 i = 0; i < collateralArr.length;) {
papr/src/PaprController.sol::370 => for (uint256 i = 0; i < collateralConfigs.length;) {
```





### [G04] Not using the named return variables when a function returns, wastes deployment gas


#### Findings:
```
papr/src/NFTEDA/NFTEDA.sol::31 => return _auctionCurrentPrice(id, startTime, auction);
papr/src/PaprController.sol::395 => return _maxDebt(totalCollateraValue, _target);
papr/src/PaprController.sol::398 => return _maxDebt(totalCollateraValue, newTarget());
papr/src/UniswapOracleFundingRateController.sol::68 => return _newTarget(latestTwapTick, _target);
papr/src/UniswapOracleFundingRateController.sol::74 => return _mark(_lastTwapTick);
papr/src/UniswapOracleFundingRateController.sol::77 => return _mark(latestTwapTick);
```



### [G05] Using `> 0` costs more gas than `!= 0` when used on a `uint` in a `require()` statement


#### Findings:
```
papr/src/PaprController.sol::170 => if (request.swapParams.minOut > 0) {
papr/src/PaprController.sol::245 => require(amount1Delta > 0); // swaps entirely within 0-liquidity regions are not supported
```



### [G06] It costs more gas to initialize variables to zero than to let the default of zero be applied


#### Findings:
```
papr/src/PaprController.sol::99 => for (uint256 i = 0; i < collateralArr.length;) {
papr/src/PaprController.sol::118 => for (uint256 i = 0; i < collateralArr.length;) {
papr/src/PaprController.sol::188 => bool hasFee = params.swapFeeBips != 0;
papr/src/PaprController.sol::213 => bool hasFee = params.swapFeeBips != 0;
papr/src/PaprController.sol::273 => bool isLastCollateral = collateralValueCached == 0;
papr/src/PaprController.sol::370 => for (uint256 i = 0; i < collateralConfigs.length;) {
papr/src/PaprController.sol::500 => bool hasFee = params.swapFeeBips != 0;
papr/src/PaprController.sol::526 => _vaultInfo[auction.nftOwner][auction.auctionAssetContract].latestAuctionStartTime = 0;
papr/src/libraries/OracleLibrary.sol::58 => secondAgos[0] = 0;
```




### [G07] Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead

#### Impact
> When using elements that are smaller than 32 bytes, your 
contract’s gas usage may be higher. This is because the EVM operates on 
32 bytes at a time. Therefore, if the element is smaller than that, the 
EVM must use more operations in order to reduce the size of the element 
from 32 bytes to the desired size.
> 

[https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html](https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html)
Use a larger size then downcast where needed

#### Findings:
```
papr/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol::12 => uint96 startTime;
papr/src/PaprController.sol::83 => uint160 initSqrtRatio;
papr/src/PaprController.sol::325 => info.latestAuctionStartTime = uint40(block.timestamp);
papr/src/PaprController.sol::436 => uint16 newCount;
papr/src/PaprController.sol::475 => _vaultInfo[account][asset].debt = uint200(newDebt);
papr/src/PaprController.sol::487 => _vaultInfo[account][asset].debt = uint200(_vaultInfo[account][asset].debt - amount);
papr/src/ReservoirOracleUnderwriter.sol::23 => uint8 v;
papr/src/UniswapOracleFundingRateController.sol::29 => uint128 internal _target;
papr/src/UniswapOracleFundingRateController.sol::31 => uint48 internal _lastUpdated;
papr/src/UniswapOracleFundingRateController.sol::55 => _lastUpdated = uint48(block.timestamp);
papr/src/UniswapOracleFundingRateController.sol::100 => _lastUpdated = uint48(block.timestamp);
papr/src/UniswapOracleFundingRateController.sol::145 => _lastCumulativeTick, tickCumulative, int56(uint56(block.timestamp - _lastUpdated))
papr/src/interfaces/IPaprController.sol::21 => uint16 count;
papr/src/interfaces/IPaprController.sol::23 => uint40 latestAuctionStartTime;
papr/src/interfaces/IPaprController.sol::25 => uint200 debt;
papr/src/interfaces/IPaprController.sol::37 => uint160 sqrtPriceLimitX96;
papr/src/libraries/OracleLibrary.sol::10 => function getQuoteAtTick(int24 tick, uint128 baseAmount, address baseToken, address quoteToken)
papr/src/libraries/OracleLibrary.sol::16 => uint160 sqrtRatioX96 = TickMath.getSqrtRatioAtTick(tick);
papr/src/libraries/OracleLibrary.sol::19 => if (sqrtRatioX96 <= type(uint128).max) {
papr/src/libraries/OracleLibrary.sol::57 => uint32[] memory secondAgos = new uint32[](1);
papr/src/libraries/PoolAddress.sol::14 => uint24 fee;
papr/src/libraries/PoolAddress.sol::22 => function getPoolKey(address tokenA, address tokenB, uint24 fee) internal pure returns (PoolKey memory) {
papr/src/libraries/PoolAddress.sol::34 => uint160(
papr/src/libraries/UniswapHelpers.sol::37 => uint160 sqrtPriceLimitX96,
papr/src/libraries/UniswapHelpers.sol::69 => function deployAndInitPool(address tokenA, address tokenB, uint24 feeTier, uint160 sqrtRatio)
papr/src/libraries/UniswapHelpers.sol::111 => return TickMath.getSqrtRatioAtTick(TickMath.getTickAtSqrtRatio(uint160((token1ONE << 96) / token0ONE)) / 2);
```



### [G08] Duplicated `require()`/`revert()` checks should be refactored to a modifier or function



#### Findings:
```
papr/src/NFTEDA/NFTEDA.sol::28 => revert InvalidAuction();
```



### [G09] Use custom errors rather than `revert()`/`require()` strings to save deployment gas


#### Findings:
```
papr/src/PaprController.sol::236 => revert("wrong caller");
papr/src/libraries/OracleLibrary.sol::39 => require(twapDuration != 0, "BP");
```



### [G10] Functions guaranteed to revert when called by normal users can be marked `payable`

#### Impact
If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.
#### Findings:
```
papr/src/PaprController.sol::350 => function setPool(address _pool) external override onlyOwner {
papr/src/PaprController.sol::355 => function setFundingPeriod(uint256 _fundingPeriod) external override onlyOwner {
papr/src/PaprController.sol::360 => function setLiquidationsLocked(bool locked) external override onlyOwner {
papr/src/PaprController.sol::382 => function sendPaprFromAuctionFees(address to, uint256 amount) external override onlyOwner {
papr/src/PaprController.sol::386 => function burnPaprFromAuctionFees(uint256 amount) external override onlyOwner {
papr/src/PaprToken.sol::24 => function mint(address to, uint256 amount) external onlyController {
papr/src/PaprToken.sol::28 => function burn(address account, uint256 amount) external onlyController {
```


### [G11] Bitshift for divide by 2

#### Impact
When multiplying or dividing by a power of two, it is cheaper to bitshift than to use standard math operations. 
#### Findings:
```
papr/src/libraries/UniswapHelpers.sol::111 => return TickMath.getSqrtRatioAtTick(TickMath.getTickAtSqrtRatio(uint160((token1ONE << 96) / token0ONE)) / 2);
```



### [G12] Amounts should be checked for 0 before calling a transfer

#### Impact
Checking non-zero transfer values can avoid an expensive external call and save gas.
#### Findings:
```
papr/src/PaprController.sol::101 => collateralArr[i].addr.transferFrom(msg.sender, address(this), collateralArr[i].id);
papr/src/PaprController.sol::202 => underlying.transfer(params.swapFeeTo, fee);
papr/src/PaprController.sol::203 => underlying.transfer(proceedsTo, amountOut - fee);
papr/src/PaprController.sol::226 => underlying.transfer(params.swapFeeTo, amountIn * params.swapFeeBips / BIPS_ONE);
papr/src/PaprController.sol::514 => underlying.transfer(params.swapFeeTo, fee);
papr/src/PaprController.sol::515 => underlying.transfer(proceedsTo, amountOut - fee);
papr/src/PaprController.sol::546 => papr.transfer(auction.nftOwner, totalOwed - debtCached);
```




### [G13] Using `bools` for storage incurs overhead


#### Findings:
```
papr/src/PaprController.sol::32 => bool public override liquidationsLocked;
papr/src/PaprController.sol::35 => bool public immutable override token0IsUnderlying;
papr/src/PaprController.sol::60 => mapping(address => bool) public override isAllowed;
```


### [G14] Using `private` rather than `public` for constants, saves gas

#### Impact
If needed, the value can be read from the verified contract source 
code. Savings are due to the compiler not having to create non-payable getter functions for deployment call data, and not adding another entry to the method ID table
#### Findings:
```
papr/src/PaprController.sol::30 => uint256 public constant BIPS_ONE = 1e4;
```




### [G15] `abi.encode()` is less efficient than abi.encodePacked()


#### Findings:
```
papr/src/NFTEDA/NFTEDA.sol::36 => return uint256(keccak256(abi.encode(auction)));
papr/src/PaprController.sol::197 => abi.encode(msg.sender, collateralAsset, oracleInfo)
papr/src/PaprController.sol::222 => abi.encode(msg.sender)
papr/src/PaprController.sol::509 => abi.encode(account, collateralAsset, oracleInfo)
papr/src/ReservoirOracleUnderwriter.sol::74 => abi.encode(
papr/src/ReservoirOracleUnderwriter.sol::93 => abi.encode(
papr/src/libraries/PoolAddress.sol::40 => keccak256(abi.encode(key.token0, key.token1, key.fee)),
```



### [G16] Multiplication/division by two should use bit shifting

#### Impact
<x> * 2 is equivalent to <x> << 1 and <x> / 2 is the same as <x> >> 1. The MUL and DIV opcodes cost 5 gas, whereas SHL and SHR only cost 3 gas
#### Findings:
```
papr/src/libraries/UniswapHelpers.sol::111 => return TickMath.getSqrtRatioAtTick(TickMath.getTickAtSqrtRatio(uint160((token1ONE << 96) / token0ONE)) / 2);
```

