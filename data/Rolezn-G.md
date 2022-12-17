## Summary<a name="Summary">

### Gas Optimizations
| |Issue|Contexts|Estimated Gas Saved|
|-|:-|:-|:-:|
| [GAS&#x2011;1](#GAS&#x2011;1) | `<x> += <y>` Costs More Gas Than `<x> = <x> + <y>` For State Variables | 2 | - |
| [GAS&#x2011;2](#GAS&#x2011;2) | `abi.encode()` is less efficient than `abi.encodepacked()` | 7 | 700 |
| [GAS&#x2011;3](#GAS&#x2011;3) | Multiplication/division By Two Should Use Bit Shifting | 1 | - |
| [GAS&#x2011;4](#GAS&#x2011;4) | Public Functions To External | 7 | - |
| [GAS&#x2011;5](#GAS&#x2011;5) | Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead | 16 | - |
| [GAS&#x2011;6](#GAS&#x2011;6) | Optimize names to save gas | 5 | 110 |
| [GAS&#x2011;7](#GAS&#x2011;7) | `internal` functions only called once can be inlined to save gas | 22 | - |
| [GAS&#x2011;8](#GAS&#x2011;8) | Setting the `constructor` to `payable` | 5 | 65 |
| [GAS&#x2011;9](#GAS&#x2011;9) | Functions guaranteed to revert when called by normal users can be marked `payable` | 8 | 168 |
| [GAS&#x2011;10](#GAS&#x2011;10) | Using `unchecked` blocks to save gas | 1 | 136 |
| [GAS&#x2011;11](#GAS&#x2011;11) | Use solidity version 0.8.17 to gain some gas boost | 10 | 880 |
| [GAS&#x2011;12](#GAS&#x2011;12) | State variables only set in the `constructor` should be declared immutable | 2 | 2000 |

Total: 86 contexts over 12 issues

## Gas Optimizations

### <a href="#Summary">[GAS&#x2011;1]</a><a name="GAS&#x2011;1"> `<x> += <y>` Costs More Gas Than `<x> = <x> + <y>` For State Variables

#### <ins>Proof Of Concept</ins>


```solidity
326: info.count -= 1;

```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L326

```solidity
419: _vaultInfo[account][collateral.addr].count += 1;

```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L419




### <a href="#Summary">[GAS&#x2011;2]</a><a name="GAS&#x2011;2"> `abi.encode()` is less efficient than `abi.encodepacked()`

See for more information: https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison 

#### <ins>Proof Of Concept</ins>


```solidity
197: abi.encode(msg.sender, collateralAsset, oracleInfo)
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L197

```solidity
222: abi.encode(msg.sender)
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L222

```solidity
509: abi.encode(account, collateralAsset, oracleInfo)
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L509

```solidity
74: abi.encode(
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/ReservoirOracleUnderwriter.sol#L74

```solidity
40: keccak256(abi.encode(key.token0, key.token1, key.fee)),
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/PoolAddress.sol#L40

```solidity
36: return uint256(keccak256(abi.encode(auction)));
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/NFTEDA.sol#L36





### <a href="#Summary">[GAS&#x2011;3]</a><a name="GAS&#x2011;3"> Multiplication/division By Two Should Use Bit Shifting

<x> * 2 is equivalent to <x> << 1 and <x> / 2 is the same as <x> >> 1. The MUL and DIV opcodes cost 5 gas, whereas SHL and SHR only cost 3 gas

#### <ins>Proof Of Concept</ins>


```solidity
111: return TickMath.getSqrtRatioAtTick(TickMath.getTickAtSqrtRatio(uint160((token1ONE << 96) / token0ONE)) / 2);
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/UniswapHelpers.sol#L111





### <a href="#Summary">[GAS&#x2011;4]</a><a name="GAS&#x2011;4"> Public Functions To External

The following functions could be set external to save gas and improve code quality.
External call cost is less expensive than of public functions.

#### <ins>Proof Of Concept</ins>


```solidity
function updateTarget() public override returns (uint256 nTarget) {
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L45

```solidity
function newTarget() public view override returns (uint256) {
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L63

```solidity
function mark() public view returns (uint256) {
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L72

```solidity
function auctionCurrentPrice(INFTEDA.Auction calldata auction) public view virtual returns (uint256) {
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/NFTEDA.sol#L24

```solidity
function auctionID(INFTEDA.Auction memory auction) public pure virtual returns (uint256) {
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/NFTEDA.sol#L35

```solidity
function auctionStartTime(uint256 id) public view virtual returns (uint256);
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/NFTEDA.sol#L40

```solidity
function auctionStartTime(uint256 id) public view virtual override returns (uint256) {
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol#L38






### <a href="#Summary">[GAS&#x2011;5]</a><a name="GAS&#x2011;5"> Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contract's gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html
Each operation involving a `uint8` costs an extra 22-28 gas (depending on whether the other operand is also a variable of type `uint8`) as compared to ones involving `uint256`, due to the compiler having to clear the higher bits of the memory word before operating on the `uint8`, as well as the associated stack operations of doing so. Use a larger size then downcast where needed

#### <ins>Proof Of Concept</ins>


```solidity
83: uint160 initSqrtRatio;
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L83

```solidity
436: uint16 newCount;
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L436

```solidity
23: uint8 v;
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/ReservoirOracleUnderwriter.sol#L23

```solidity
29: uint128 internal _target;
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L29

```solidity
30: int56 internal _lastCumulativeTick;
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L30

```solidity
31: uint48 internal _lastUpdated;
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L31

```solidity
32: int24 internal _lastTwapTick;
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L32

```solidity
21: uint16 count;
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/interfaces/IPaprController.sol#L21

```solidity
23: uint40 latestAuctionStartTime;
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/interfaces/IPaprController.sol#L23

```solidity
25: uint200 debt;
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/interfaces/IPaprController.sol#L25

```solidity
37: uint160 sqrtPriceLimitX96;
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/interfaces/IPaprController.sol#L37

```solidity
16: uint160 sqrtRatioX96 = TickMath.getSqrtRatioAtTick(tick);
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/OracleLibrary.sol#L16

```solidity
42: int56 delta = endTick - startTick;
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/OracleLibrary.sol#L42

```solidity
57: uint32[] memory secondAgos = new uint32[](1);
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/OracleLibrary.sol#L57

```solidity
14: uint24 fee;
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/PoolAddress.sol#L14

```solidity
12: uint96 startTime;
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol#L12






### <a href="#Summary">[GAS&#x2011;6]</a><a name="GAS&#x2011;6"> Optimize names to save gas

Contracts most called functions could simply save gas by function ordering via Method ID. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because 22 gas are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions. 

See more <a href="https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92">here</a>

#### <ins>Proof Of Concept</ins>

```solidity
File: PaprToken.sol
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprToken.sol

```solidity
File: ReservoirOracleUnderwriter.sol
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/ReservoirOracleUnderwriter.sol

```solidity
File: UniswapOracleFundingRateController.sol
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol

```solidity
File: NFTEDA.sol
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/NFTEDA.sol

```solidity
File: NFTEDAStarterIncentive.sol
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol



#### <ins>Recommended Mitigation Steps</ins>
Find a lower method ID name for the most called functions for example Call() vs. Call1() is cheaper by 22 gas
For example, the function IDs in the Gauge.sol contract will be the most used; A lower method ID may be given.







### <a href="#Summary">[GAS&#x2011;7]</a><a name="GAS&#x2011;7"> `internal` functions only called once can be inlined to save gas

#### <ins>Proof Of Concept</ins>

```solidity
424: function _removeCollateral

```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L424

```solidity
493: function _increaseDebtAndSell

```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L493

```solidity
519: function _purchaseNFTAndUpdateVaultIfNeeded

```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L519

```solidity
532: function _handleExcess

```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L532

```solidity
95: function _init

```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L95

```solidity
111: function _setPool

```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L111

```solidity
122: function _setFundingPeriod

```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L122

```solidity
156: function _multiplier

```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L156

```solidity
56: function latestCumulativeTick

```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/OracleLibrary.sol#L56

```solidity
22: function getPoolKey

```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/PoolAddress.sol#L22

```solidity
31: function computeAddress

```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/PoolAddress.sol#L31

```solidity
31: function swap

```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/UniswapHelpers.sol#L31

```solidity
69: function deployAndInitPool

```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/UniswapHelpers.sol#L69

```solidity
82: function poolCurrentTick

```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/UniswapHelpers.sol#L82

```solidity
92: function poolsHaveSameTokens

```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/UniswapHelpers.sol#L92

```solidity
100: function isUniswapPool

```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/UniswapHelpers.sol#L100

```solidity
110: function oneToOneSqrtRatio

```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/UniswapHelpers.sol#L110

```solidity
101: function _setAuctionStartTime

```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/NFTEDA.sol#L101

```solidity
43: function _setAuctionStartTime

```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol#L43

```solidity
48: function _clearAuctionState

```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol#L48

```solidity
72: function _setCreatorDiscount

```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol#L72

```solidity
10: function currentPrice

```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/libraries/EDAPrice.sol#L10






### <a href="#Summary">[GAS&#x2011;8]</a><a name="GAS&#x2011;8"> Setting the `constructor` to `payable`

Saves ~13 gas per instance

#### <ins>Proof Of Concept</ins>

```solidity
67: constructor(
        string memory name,
        string memory symbol,
        uint256 _maxLTV,
        uint256 indexMarkRatioMax,
        uint256 indexMarkRatioMin,
        ERC20 underlying,
        address oracleSigner
    )
        NFTEDAStarterIncentive(1e17)
        UniswapOracleFundingRateController(underlying, new PaprToken(name, symbol), indexMarkRatioMax, indexMarkRatioMin)
        ReservoirOracleUnderwriter(oracleSigner, address(underlying))
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L67

```solidity
18: constructor(string memory name, string memory symbol)
        ERC20(string.concat("papr ", name), string.concat("papr", symbol), 18)
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprToken.sol#L18

```solidity
51: constructor(address _oracleSigner, address _quoteCurrency)
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/ReservoirOracleUnderwriter.sol#L51

```solidity
34: constructor(ERC20 _underlying, ERC20 _papr, uint256 _targetMarkRatioMax, uint256 _targetMarkRatioMin)
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L34

```solidity
33: constructor(uint256 _auctionCreatorDiscountPercentWad)
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol#L33





### <a href="#Summary">[GAS&#x2011;9]</a><a name="GAS&#x2011;9"> Functions guaranteed to revert when called by normal users can be marked `payable`

If a function modifier or require such as onlyOwner/onlyX is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

#### <ins>Proof Of Concept</ins>

```solidity
350: function setPool(address _pool) external override onlyOwner {

```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L350

```solidity
355: function setFundingPeriod(uint256 _fundingPeriod) external override onlyOwner {

```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L355

```solidity
360: function setLiquidationsLocked(bool locked) external override onlyOwner {

```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L360

```solidity
365: function setAllowedCollateral(IPaprController.CollateralAllowedConfig[] calldata collateralConfigs)
        external
        override
        onlyOwner
    {

```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L365

```solidity
382: function sendPaprFromAuctionFees(address to, uint256 amount) external override onlyOwner {

```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L382

```solidity
386: function burnPaprFromAuctionFees(uint256 amount) external override onlyOwner {

```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L386

```solidity
24: function mint(address to, uint256 amount) external onlyController {

```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprToken.sol#L24

```solidity
28: function burn(address account, uint256 amount) external onlyController {

```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprToken.sol#L28



#### <ins>Recommended Mitigation Steps</ins>
Functions guaranteed to revert when called by normal users can be marked payable.



### <a href="#Summary">[GAS&#x2011;10]</a><a name="GAS&#x2011;10"> Using `unchecked` blocks to save gas

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an `unchecked` block

#### <ins>Proof Of Concept</ins>

```solidity
42: int56 delta = endTick - startTick;

```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/OracleLibrary.sol#L42





### <a href="#Summary">[GAS&#x2011;11]</a><a name="GAS&#x2011;11"> Use solidity version 0.8.17 to gain some gas boost

Upgrade to the latest solidity version 0.8.17 to get additional gas savings.

#### <ins>Proof Of Concept</ins>


```solidity
pragma solidity >=0.8.0;
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/interfaces/IFundingRateController.sol#L2

```solidity
pragma solidity >=0.8.0;
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/interfaces/IPaprController.sol#L2

```solidity
pragma solidity >=0.8.0;
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/interfaces/IUniswapOracleFundingRateController.sol#L2

```solidity
pragma solidity >=0.8.0;
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/OracleLibrary.sol#L2

```solidity
pragma solidity >=0.8.0;
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/PoolAddress.sol#L4

```solidity
pragma solidity >=0.8.0;
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/UniswapHelpers.sol#L2

```solidity
pragma solidity >=0.8.0;
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/NFTEDA.sol#L2

```solidity
pragma solidity >=0.8.0;
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol#L2

```solidity
pragma solidity >=0.8.0;
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/interfaces/INFTEDA.sol#L2

```solidity
pragma solidity >=0.8.0;
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/libraries/EDAPrice.sol#L2


### <a href="#Summary">[GAS&#x2011;12]</a><a name="GAS&#x2011;12"> State variables only set in the `constructor` should be declared immutable

Avoids a Gsset (20000 gas) in the constructor, and replaces each Gwarmacces (100 gas) with a PUSH32 (3 gas).

#### <ins>Proof Of Concept</ins>

```solidity
26:    uint256 public auctionCreatorDiscountPercentWad;
27:    uint256 internal _pricePercentAfterDiscount;
```

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol#L26-L27

