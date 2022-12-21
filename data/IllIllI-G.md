
## Summary

### Gas Optimizations
| |Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
| [G&#x2011;01] | Avoid contract existence checks by using low level calls | 7 |  700 |
| [G&#x2011;02] | `internal` functions only called once can be inlined to save gas | 5 |  100 |
| [G&#x2011;03] | Add `unchecked {}` for subtractions where the operands cannot underflow because of a previous `require()` or `if`-statement | 1 |  85 |
| [G&#x2011;04] | `keccak256()` should only need to be called on a specific string literal once | 2 |  84 |
| [G&#x2011;05] | Optimize names to save gas | 8 |  176 |
| [G&#x2011;06] | Use a more recent version of solidity | 10 |  - |
| [G&#x2011;07] | `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too) | 1 |  5 |
| [G&#x2011;08] | Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead | 5 |  - |
| [G&#x2011;09] | Using `private` rather than `public` for constants, saves gas | 2 |  - |
| [G&#x2011;10] | Division by two should use bit shifting | 1 |  20 |
| [G&#x2011;11] | Functions guaranteed to revert when called by normal users can be marked `payable` | 8 |  168 |

Total: 50 instances over 11 issues with **1338 gas** saved

Gas totals use lower bounds of ranges and count two iterations of each `for`-loop. All values above are runtime, not deployment, values; deployment values are listed in the individual issue descriptions. The table above as well as its gas numbers do not include any of the excluded findings.





## Gas Optimizations

### [G&#x2011;01]  Avoid contract existence checks by using low level calls
Prior to 0.8.10 the compiler inserted extra code, including `EXTCODESIZE` (**100 gas**), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence

*There are 7 instances of this issue:*

```solidity
File: src/libraries/OracleLibrary.sol

/// @audit observe()
59:           (int56[] memory tickCumulatives,) = IUniswapV3Pool(pool).observe(secondAgos);

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/libraries/OracleLibrary.sol#L59

```solidity
File: src/libraries/UniswapHelpers.sol

/// @audit swap()
40:           (int256 amount0, int256 amount1) = IUniswapV3Pool(pool).swap(

/// @audit slot0()
83:           (, int24 tick,,,,,) = IUniswapV3Pool(pool).slot0();

/// @audit token0()
/// @audit token0()
93:           return IUniswapV3Pool(pool1).token0() == IUniswapV3Pool(pool2).token0()

/// @audit token1()
/// @audit token1()
94:               && IUniswapV3Pool(pool1).token1() == IUniswapV3Pool(pool2).token1();

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/libraries/UniswapHelpers.sol#L40

### [G&#x2011;02]  `internal` functions only called once can be inlined to save gas
Not inlining costs **20 to 40 gas** because of two extra `JUMP` instructions and additional stack operations needed for function calls.

*There are 5 instances of this issue:*

```solidity
File: src/PaprController.sol

424       function _removeCollateral(
425           address sendTo,
426           IPaprController.Collateral calldata collateral,
427           uint256 oraclePrice,
428:          uint256 cachedTarget

493       function _increaseDebtAndSell(
494           address account,
495           address proceedsTo,
496           ERC721 collateralAsset,
497           IPaprController.SwapParams memory params,
498           ReservoirOracleUnderwriter.OracleInfo memory oracleInfo
499:      ) internal returns (uint256 amountOut) {

519       function _purchaseNFTAndUpdateVaultIfNeeded(Auction calldata auction, uint256 maxPrice, address sendTo)
520           internal
521:          returns (uint256)

532       function _handleExcess(uint256 excess, uint256 neededToSaveVault, uint256 debtCached, Auction calldata auction)
533           internal
534:          returns (uint256 remaining)

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/PaprController.sol#L424-L428

```solidity
File: src/UniswapOracleFundingRateController.sol

156:      function _multiplier(uint256 _mark_, uint256 cachedTarget) internal view returns (uint256) {

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/UniswapOracleFundingRateController.sol#L156

### [G&#x2011;03]  Add `unchecked {}` for subtractions where the operands cannot underflow because of a previous `require()` or `if`-statement
`require(a <= b); x = b - a` => `require(a <= b); unchecked { x = b - a }`

*There is 1 instance of this issue:*

```solidity
File: src/PaprController.sol

/// @audit if-condition on line 542
546:              papr.transfer(auction.nftOwner, totalOwed - debtCached);

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/PaprController.sol#L546

### [G&#x2011;04]  `keccak256()` should only need to be called on a specific string literal once
It should be saved to an immutable variable, and the variable used instead. If the hash is being used as a part of a function selector, the cast to `bytes4` should also only be done once

*There are 2 instances of this issue:*

```solidity
File: src/ReservoirOracleUnderwriter.sol

75:                               keccak256("Message(bytes32 id,bytes payload,uint256 timestamp)"),

94:                   keccak256("ContractWideCollectionPrice(uint8 kind,uint256 twapSeconds,address contract)"),

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/ReservoirOracleUnderwriter.sol#L75

### [G&#x2011;05]  Optimize names to save gas
`public`/`external` function names and `public` member variable names can be optimized to save gas. See [this](https://gist.github.com/IllIllI000/a5d8b486a8259f9f77891a919febd1a9) link for an example of how it works. Below are the interfaces/abstract contracts that can be optimized so that the most frequently-called functions use the least amount of gas possible during method lookup. Method IDs that have two leading zero bytes can save **128 gas** each during deployment, and renaming functions to have lower method IDs will save **22 gas** per call, [per sorted position shifted](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92)

*There are 8 instances of this issue:*

```solidity
File: src/interfaces/IFundingRateController.sol

/// @audit updateTarget(), lastUpdated(), target(), newTarget(), mark(), papr(), fundingPeriod()
6:    interface IFundingRateController {

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/interfaces/IFundingRateController.sol#L6

```solidity
File: src/interfaces/IPaprController.sol

/// @audit addCollateral(), removeCollateral(), increaseDebt(), reduceDebt(), increaseDebtAndSell(), buyAndReduceDebt(), purchaseLiquidationAuctionNFT(), startLiquidationAuction(), setPool(), setFundingPeriod(), setLiquidationsLocked(), setAllowedCollateral(), sendPaprFromAuctionFees(), burnPaprFromAuctionFees(), collateralOwner(), isAllowed(), liquidationsLocked(), token0IsUnderlying(), maxLTV(), liquidationAuctionMinSpacing(), perPeriodAuctionDecayWAD(), auctionDecayPeriod(), auctionStartPriceMultiplier(), liquidationPenaltyBips(), maxDebt(), vaultInfo()
9:    interface IPaprController {

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/interfaces/IPaprController.sol#L9

```solidity
File: src/interfaces/IUniswapOracleFundingRateController.sol

/// @audit pool()
6:    interface IUniswapOracleFundingRateController is IFundingRateController {

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/interfaces/IUniswapOracleFundingRateController.sol#L6

```solidity
File: src/NFTEDA/interfaces/INFTEDA.sol

/// @audit auctionCurrentPrice(), auctionID(), auctionStartTime()
7:    interface INFTEDA {

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/NFTEDA/interfaces/INFTEDA.sol#L7

```solidity
File: src/NFTEDA/NFTEDA.sol

/// @audit auctionCurrentPrice(), auctionID(), auctionStartTime()
11:   abstract contract NFTEDA is INFTEDA {

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/NFTEDA/NFTEDA.sol#L11

```solidity
File: src/PaprController.sol

/// @audit uniswapV3SwapCallback()
18:   contract PaprController is

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/PaprController.sol#L18

```solidity
File: src/ReservoirOracleUnderwriter.sol

/// @audit underwritePriceForCollateral()
7:    contract ReservoirOracleUnderwriter {

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/ReservoirOracleUnderwriter.sol#L7

```solidity
File: src/UniswapOracleFundingRateController.sol

/// @audit mark()
15:   contract UniswapOracleFundingRateController is IUniswapOracleFundingRateController {

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/UniswapOracleFundingRateController.sol#L15

### [G&#x2011;06]  Use a more recent version of solidity
Use a solidity version of at least 0.8.2 to get simple compiler automatic inlining
Use a solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads
Use a solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than `revert()/require()` strings
Use a solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value

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

### [G&#x2011;07]  `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too)
Saves **5 gas per loop**

*There is 1 instance of this issue:*

```solidity
File: src/libraries/OracleLibrary.sol

48:                   twat--;

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/libraries/OracleLibrary.sol#L48

### [G&#x2011;08]  Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead
> When using elements that are smaller than 32 bytes, your contractâ€™s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html
Each operation involving a `uint8` costs an extra [**22-28 gas**](https://gist.github.com/IllIllI000/9388d20c70f9a4632eb3ca7836f54977) (depending on whether the other operand is also a variable of type `uint8`) as compared to ones involving `uint256`, due to the compiler having to clear the higher bits of the memory word before operating on the `uint8`, as well as the associated stack operations of doing so. Use a larger size then downcast where needed

*There are 5 instances of this issue:*

```solidity
File: src/libraries/OracleLibrary.sol

/// @audit int24 twat
44:               twat = int24(delta / twapDuration);

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/libraries/OracleLibrary.sol#L44

```solidity
File: src/PaprController.sol

/// @audit uint16 newCount
438:              newCount = _vaultInfo[msg.sender][collateral.addr].count - 1;

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/PaprController.sol#L438

```solidity
File: src/UniswapOracleFundingRateController.sol

/// @audit uint48 _lastUpdated
55:           _lastUpdated = uint48(block.timestamp);

/// @audit uint48 _lastUpdated
100:          _lastUpdated = uint48(block.timestamp);

/// @audit int56 tickCumulative
143:          tickCumulative = OracleLibrary.latestCumulativeTick(pool);

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/UniswapOracleFundingRateController.sol#L55

### [G&#x2011;09]  Using `private` rather than `public` for constants, saves gas
If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178) of the values of all currently-public constants. Saves **3406-3606 gas** in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

*There are 2 instances of this issue:*

```solidity
File: src/UniswapOracleFundingRateController.sol

25:       uint256 public immutable targetMarkRatioMax;

27:       uint256 public immutable targetMarkRatioMin;

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/UniswapOracleFundingRateController.sol#L25

### [G&#x2011;10]  Division by two should use bit shifting
`<x> / 2` is the same as `<x> >> 1`. While the compiler uses the `SHR` opcode to accomplish both, the version that uses division incurs an overhead of [**20 gas**](https://gist.github.com/IllIllI000/ec0e4e6c4f52a6bca158f137a3afd4ff) due to `JUMP`s to and from a compiler utility function that introduces checks which can be avoided by using `unchecked {}` around the division by two

*There is 1 instance of this issue:*

```solidity
File: src/libraries/UniswapHelpers.sol

111:          return TickMath.getSqrtRatioAtTick(TickMath.getTickAtSqrtRatio(uint160((token1ONE << 96) / token0ONE)) / 2);

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/libraries/UniswapHelpers.sol#L111

### [G&#x2011;11]  Functions guaranteed to revert when called by normal users can be marked `payable`
If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are 
`CALLVALUE`(2),`DUP1`(3),`ISZERO`(3),`PUSH2`(3),`JUMPI`(10),`PUSH1`(3),`DUP1`(3),`REVERT`(0),`JUMPDEST`(1),`POP`(2), which costs an average of about **21 gas per call** to the function, in addition to the extra deployment cost

*There are 8 instances of this issue:*

```solidity
File: src/PaprController.sol

350:      function setPool(address _pool) external override onlyOwner {

355:      function setFundingPeriod(uint256 _fundingPeriod) external override onlyOwner {

360:      function setLiquidationsLocked(bool locked) external override onlyOwner {

365       function setAllowedCollateral(IPaprController.CollateralAllowedConfig[] calldata collateralConfigs)
366           external
367           override
368:          onlyOwner

382:      function sendPaprFromAuctionFees(address to, uint256 amount) external override onlyOwner {

386:      function burnPaprFromAuctionFees(uint256 amount) external override onlyOwner {

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/PaprController.sol#L350

```solidity
File: src/PaprToken.sol

24:       function mint(address to, uint256 amount) external onlyController {

28:       function burn(address account, uint256 amount) external onlyController {

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/PaprToken.sol#L24


___

## Excluded findings
These findings are excluded from awards calculations because there are publicly-available automated tools that find them. The valid ones appear here for completeness

## Summary

### Gas Optimizations
| |Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
| [G&#x2011;01] | Using `calldata` instead of `memory` for read-only arguments in `external` functions saves gas | 1 |  120 |
| [G&#x2011;02] | State variables should be cached in stack variables rather than re-reading them from storage | 2 |  194 |
| [G&#x2011;03] | `<array>.length` should not be looked up in every loop of a `for`-loop | 3 |  9 |
| [G&#x2011;04] | Using `bool`s for storage incurs overhead | 3 |  51300 |
| [G&#x2011;05] | Using `private` rather than `public` for constants, saves gas | 1 |  - |
| [G&#x2011;06] | Use custom errors rather than `revert()`/`require()` strings to save gas | 1 |  - |

Total: 11 instances over 6 issues with **51623 gas** saved

Gas totals use lower bounds of ranges and count two iterations of each `for`-loop. All values above are runtime, not deployment, values; deployment values are listed in the individual issue descriptions. The table above as well as its gas numbers do not include any of the excluded findings.





## Gas Optimizations

### [G&#x2011;01]  Using `calldata` instead of `memory` for read-only arguments in `external` functions saves gas
When a function with a `memory` array is called externally, the `abi.decode()` step has to use a for-loop to copy each index of the `calldata` to the `memory` index. **Each iteration of this for-loop costs at least 60 gas** (i.e. `60 * <mem_array>.length`). Using `calldata` directly, obliviates the need for such a loop in the contract code and runtime execution. Note that even if an interface defines a function as having `memory` arguments, it's still valid for implementation contracs to use `calldata` arguments instead. 

If the array is passed to an `internal` function which passes the array to another internal function where the array is modified and therefore `memory` is used in the `external` call, it's still more gass-efficient to use `calldata` when the `external` function uses modifiers, since the modifiers may prevent the internal functions from being called. Structs have the same overhead as an array of length one

Note that I've also flagged instances where the function is `public` but can be marked as `external` since it's not called by the contract, and cases where a constructor is involved

*There is 1 instance of this issue:*

```solidity
File: src/ReservoirOracleUnderwriter.sol

/// @audit oracleInfo - (valid but excluded finding)
64        function underwritePriceForCollateral(ERC721 asset, PriceKind priceKind, OracleInfo memory oracleInfo)
65            public
66:           returns (uint256)

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/ReservoirOracleUnderwriter.sol#L64-L66

### [G&#x2011;02]  State variables should be cached in stack variables rather than re-reading them from storage
The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (**100 gas**) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

*There are 2 instances of this issue:*

```solidity
File: src/UniswapOracleFundingRateController.sol

/// @audit pool on line 102 - (valid but excluded finding)
103:          _lastTwapTick = UniswapHelpers.poolCurrentTick(pool);

/// @audit pool on line 112 - (valid but excluded finding)
112:          if (pool != address(0) && !UniswapHelpers.poolsHaveSameTokens(pool, _pool)) revert PoolTokensDoNotMatch();

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/UniswapOracleFundingRateController.sol#L103

### [G&#x2011;03]  `<array>.length` should not be looked up in every loop of a `for`-loop
The overheads outlined below are _PER LOOP_, excluding the first loop
* storage arrays incur a Gwarmaccess (**100 gas**)
* memory arrays use `MLOAD` (**3 gas**)
* calldata arrays use `CALLDATALOAD` (**3 gas**)

Caching the length changes each of these to a `DUP<N>` (**3 gas**), and gets rid of the extra `DUP<N>` needed to store the stack offset

*There are 3 instances of this issue:*

```solidity
File: src/PaprController.sol

/// @audit (valid but excluded finding)
99:           for (uint256 i = 0; i < collateralArr.length;) {

/// @audit (valid but excluded finding)
118:          for (uint256 i = 0; i < collateralArr.length;) {

/// @audit (valid but excluded finding)
370:          for (uint256 i = 0; i < collateralConfigs.length;) {

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/PaprController.sol#L99

### [G&#x2011;04]  Using `bool`s for storage incurs overhead
```solidity
    // Booleans are more expensive than uint256 or any type that takes up a full
    // word because each write operation emits an extra SLOAD to first read the
    // slot's contents, replace the bits taken up by the boolean, and then write
    // back. This is the compiler's defense against contract upgrades and
    // pointer aliasing, and it cannot be disabled.
```
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27
Use `uint256(1)` and `uint256(2)` for true/false to avoid a Gwarmaccess (**[100 gas](https://gist.github.com/IllIllI000/1b70014db712f8572a72378321250058)**) for the extra SLOAD, and to avoid Gsset (**20000 gas**) when changing from `false` to `true`, after having been `true` in the past

*There are 3 instances of this issue:*

```solidity
File: src/PaprController.sol

/// @audit (valid but excluded finding)
32:       bool public override liquidationsLocked;

/// @audit (valid but excluded finding)
35:       bool public immutable override token0IsUnderlying;

/// @audit (valid but excluded finding)
60:       mapping(address => bool) public override isAllowed;

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/PaprController.sol#L32

### [G&#x2011;05]  Using `private` rather than `public` for constants, saves gas
If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178) of the values of all currently-public constants. Saves **3406-3606 gas** in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

*There is 1 instance of this issue:*

```solidity
File: src/PaprController.sol

/// @audit (valid but excluded finding)
30:       uint256 public constant BIPS_ONE = 1e4;

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/PaprController.sol#L30

### [G&#x2011;06]  Use custom errors rather than `revert()`/`require()` strings to save gas
Custom errors are available from solidity version 0.8.4. Custom errors save [**~50 gas**](https://gist.github.com/IllIllI000/ad1bd0d29a0101b25e57c293b4b0c746) each time they're hit by [avoiding having to allocate and store the revert string](https://blog.soliditylang.org/2021/04/21/custom-errors/#errors-in-depth). Not defining the strings also save deployment gas

*There is 1 instance of this issue:*

```solidity
File: src/libraries/OracleLibrary.sol

/// @audit (valid but excluded finding)
39:           require(twapDuration != 0, "BP");

```
https://github.com/with-backed/papr/blob/1933da2e38ff9d47c17e2749d6088bbbd40bfa68/src/libraries/OracleLibrary.sol#L39

