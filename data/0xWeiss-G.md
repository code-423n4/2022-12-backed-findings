Cheaper to use abi.encode than string.concat
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprToken.sol#L19
##########reformulate first sentence


Instead of immutable change it for constant because you are not initializing the values in the constructor:
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L41-L54



###sketchy check more this transfer https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L546



####uint packing https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/ReservoirOracleUnderwriter.sol#L35-L38




### Don't Initialize Variables with Default Value

#### Impact
Issue Information: [G001](https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md#g001---dont-initialize-variables-with-default-value)

#### Findings:
```
examples/src/PaprController.sol::99 => for (uint256 i = 0; i < collateralArr.length;) {
examples/src/PaprController.sol::118 => for (uint256 i = 0; i < collateralArr.length;) {
examples/src/PaprController.sol::370 => for (uint256 i = 0; i < collateralConfigs.length;) {
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)

### Cache Array Length Outside of Loop

#### Impact
Issue Information: [G002](https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md#g002---cache-array-length-outside-of-loop)

#### Findings:
```
examples/src/PaprController.sol::99 => for (uint256 i = 0; i < collateralArr.length;) {
examples/src/PaprController.sol::118 => for (uint256 i = 0; i < collateralArr.length;) {
examples/src/PaprController.sol::370 => for (uint256 i = 0; i < collateralConfigs.length;) {
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)

### Use != 0 instead of > 0 for Unsigned Integer Comparison

#### Impact
Issue Information: [G003](https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md#g003---use--0-instead-of--0-for-unsigned-integer-comparison)

#### Findings:
```
examples/src/PaprController.sol::170 => if (request.swapParams.minOut > 0) {
examples/src/PaprController.sol::172 => } else if (request.debt > 0) {
examples/src/PaprController.sol::241 => if (amount0Delta > 0) {
examples/src/PaprController.sol::245 => require(amount1Delta > 0); // swaps entirely within 0-liquidity regions are not supported
examples/src/PaprController.sol::283 => if (excess > 0) {
examples/src/UniswapOracleFundingRateController.sol::171 => // safe to cast because targetMarkRatio > 0
examples/src/interfaces/IPaprController.sol::48 => /// @dev debt is ignored in favor of `swapParams.amount` of minOut > 0
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)

### Use immutable for OpenZeppelin AccessControl's Roles Declarations

#### Impact
Issue Information: [G006](https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md#g006---use-immutable-for-openzeppelin-accesscontrols-roles-declarations)

#### Findings:
```
examples/src/NFTEDA/NFTEDA.sol::36 => return uint256(keccak256(abi.encode(auction)));
examples/src/ReservoirOracleUnderwriter.sol::69 => keccak256(
examples/src/ReservoirOracleUnderwriter.sol::73 => keccak256(
examples/src/ReservoirOracleUnderwriter.sol::75 => keccak256("Message(bytes32 id,bytes payload,uint256 timestamp)"),
examples/src/ReservoirOracleUnderwriter.sol::77 => keccak256(oracleInfo.message.payload),
examples/src/ReservoirOracleUnderwriter.sol::92 => bytes32 expectedId = keccak256(
examples/src/ReservoirOracleUnderwriter.sol::94 => keccak256("ContractWideCollectionPrice(uint8 kind,uint256 twapSeconds,address contract)"),
examples/src/libraries/PoolAddress.sol::36 => keccak256(
examples/src/libraries/PoolAddress.sol::40 => keccak256(abi.encode(key.token0, key.token1, key.fee)),
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)


### Use Shift Right/Left instead of Division/Multiplication if possible

#### Impact
Issue Information: [G008](https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md/#g008---use-shift-rightleft-instead-of-divisionmultiplication-if-possible)

#### Findings:
```
examples/src/libraries/UniswapHelpers.sol::111 => return TickMath.getSqrtRatioAtTick(TickMath.getTickAtSqrtRatio(uint160((token1ONE << 96) / token0ONE)) / 2);
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)