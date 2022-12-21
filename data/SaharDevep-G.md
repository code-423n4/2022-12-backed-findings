# c4udit Report


## Issues found

### Don't Initialize Variables with Default Value

#### Impact
Issue Information: [G001](https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md#g001---dont-initialize-variables-with-default-value)

#### Findings:
```
src\PaprController.sol::99 => for (uint256 i = 0; i < collateralArr.length;) {
src\PaprController.sol::118 => for (uint256 i = 0; i < collateralArr.length;) {
src\PaprController.sol::370 => for (uint256 i = 0; i < collateralConfigs.length;) {
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)

### Cache Array Length Outside of Loop

#### Impact
Issue Information: [G002](https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md#g002---cache-array-length-outside-of-loop)

#### Findings:
```
src\PaprController.sol::99 => for (uint256 i = 0; i < collateralArr.length;) {
src\PaprController.sol::118 => for (uint256 i = 0; i < collateralArr.length;) {
src\PaprController.sol::370 => for (uint256 i = 0; i < collateralConfigs.length;) {
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)

### Use != 0 instead of > 0 for Unsigned Integer Comparison

#### Impact
Issue Information: [G003](https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md#g003---use--0-instead-of--0-for-unsigned-integer-comparison)

#### Findings:
```
src\PaprController.sol::170 => if (request.swapParams.minOut > 0) {
src\PaprController.sol::172 => } else if (request.debt > 0) {
src\PaprController.sol::241 => if (amount0Delta > 0) {
src\PaprController.sol::245 => require(amount1Delta > 0); // swaps entirely within 0-liquidity regions are not supported
src\PaprController.sol::283 => if (excess > 0) {
src\UniswapOracleFundingRateController.sol::171 => // safe to cast because targetMarkRatio > 0
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)


### Long Revert Strings

#### Impact
Issue Information: [G004](https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md#g007---long-revert-strings)

#### Findings:
```
src\EDAPrice.sol::4 => import {FixedPointMathLib} from "solmate/utils/FixedPointMathLib.sol";
src\EDAPrice.sol::5 => import {SafeCast} from "v3-core/contracts/libraries/SafeCast.sol";
src\NFTEDAStarterIncentive.sol::4 => import {FixedPointMathLib} from "solmate/utils/FixedPointMathLib.sol";
src\OracleLibrary.sol::4 => import {IUniswapV3Pool} from "v3-core/contracts/interfaces/IUniswapV3Pool.sol";
src\PaprController.sol::6 => import {SafeTransferLib} from "solmate/utils/SafeTransferLib.sol";
src\PaprController.sol::7 => import {FixedPointMathLib} from "solmate/utils/FixedPointMathLib.sol";
src\PaprController.sol::8 => import {INFTEDA, NFTEDAStarterIncentive} from "./NFTEDA/extensions/NFTEDAStarterIncentive.sol";
src\PaprController.sol::9 => import {Ownable2Step} from "openzeppelin-contracts/access/Ownable2Step.sol";
src\PaprController.sol::13 => import {UniswapOracleFundingRateController} from "./UniswapOracleFundingRateController.sol";
src\PaprToken.sol::19 => ERC20(string.concat("papr ", name), string.concat("papr", symbol), 18)
src\ReservoirOracleUnderwriter.sol::75 => keccak256("Message(bytes32 id,bytes payload,uint256 timestamp)"),
src\ReservoirOracleUnderwriter.sol::94 => keccak256("ContractWideCollectionPrice(uint8 kind,uint256 twapSeconds,address contract)"),
src\UniswapHelpers.sol::4 => import {IUniswapV3Pool} from "v3-core/contracts/interfaces/IUniswapV3Pool.sol";
src\UniswapHelpers.sol::5 => import {IUniswapV3Factory} from "v3-core/contracts/interfaces/IUniswapV3Factory.sol";
src\UniswapHelpers.sol::8 => import {SafeCast} from "v3-core/contracts/libraries/SafeCast.sol";
src\UniswapOracleFundingRateController.sol::5 => import {FixedPointMathLib} from "solmate/utils/FixedPointMathLib.sol";
src\UniswapOracleFundingRateController.sol::13 => } from "./interfaces/IUniswapOracleFundingRateController.sol";
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)

### Use Shift Right/Left instead of Division/Multiplication if possible

#### Impact
Issue Information: [G005](https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md/#g008---use-shift-rightleft-instead-of-divisionmultiplication-if-possible)

#### Findings:
```
src\UniswapHelpers.sol::111 => return TickMath.getSqrtRatioAtTick(TickMath.getTickAtSqrtRatio(uint160((token1ONE << 96) / token0ONE)) / 2);
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)


### x += y COSTS MORE GAS THAN x = x + y FOR STATE VARIABLES

#### Impact
[G006]

#### Findings:
```
src\PaprController.sol::419 => _vaultInfo[account][collateral.addr].count += 1;

```

### INTERNAL FUNCTIONS ONLY CALLED ONCE CAN BE INLINED TO SAVE GAS

#### Impact
[G007]

#### Findings:
```
src\PaprController.sol::493 => function _increaseDebtAndSell(
src\PaprController.sol::499 => function _purchaseNFTAndUpdateVaultIfNeeded(Auction calldata auction, uint256 maxPrice, address sendTo)
src\PaprController.sol::532 => function _handleExcess(uint256 excess, uint256 neededToSaveVault, uint256 debtCached, Auction calldata auction)

```

### USING > 0 COSTS MORE GAS THAN != 0 WHEN USED ON A UINT IN A REQUIRE() STATEMENT

#### Impact 
[G008]

#### Findings:
```
src\PaprController.sol::245 => require(amount1Delta > 0);

```

### USING CALLDATA INSTEAD OF MEMORY FOR READ-ONLY ARGUMENTS IN EXTERNAL FUNCTIONS SAVES GAS

#### Impact
[G009]

#### Findings:
```
src\PaprController.sol::413 => function _addCollateralToVault(address account, IPaprController.Collateral memory collateral) internal {
src\PaprController.sol::461 => ReservoirOracleUnderwriter.OracleInfo memory oracleInfo
src\PaprController.sol::497 => IPaprController.SwapParams memory params,
src\PaprController.sol::498 => ReservoirOracleUnderwriter.OracleInfo memory oracleInfo
src\PaprController.sol::64 => function underwritePriceForCollateral(ERC721 asset, PriceKind priceKind, OracleInfo memory oracleInfo)

```

### USING BOOLS FOR STORAGE INCURS OVERHEAD

#### Impact
[G010]

#### Findings:
```
src\PaprController.sol::32 => bool public override liquidationsLocked;
src\PaprController.sol::33 => bool public immutable override token0IsUnderlying;

```

### USING BOOLS FOR STORAGE INCURS OVERHEAD

#### Impact
[G011]

#### Findings:
```
src\PaprController.sol::197 => abi.encode(msg.sender, collateralAsset, oracleInfo)
src\PaprController.sol::222 => abi.encode(msg.sender)
src\PaprController.sol::509 => abi.encode(account, collateralAsset, oracleInfo)
src\ReservoirOracleUnderwriter.sol::74 => abi.encode(
src\ReservoirOracleUnderwriter.sol::93 => abi.encode(

```

### USING PRIVATE RATHER THAN PUBLIC FOR CONSTANTS, SAVES GAS

#### Impact
[G012]

#### Findings:
```
src\PaprController.sol::30 => uint256 public constant BIPS_ONE = 1e4;
src\ReservoirOracleUnderwriter.sol::35 => uint256 constant TWAP_SECONDS = 30 days;
src\ReservoirOracleUnderwriter.sol::38 => uint256 constant VALID_FOR = 20 minutes;

```