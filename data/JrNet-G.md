## [G001] >= costs less gas than >

The compiler uses opcodes `GT` and `ISZERO` for solidity code that uses `>`, but only requires `LT` for `>=`, which saves 3 gas

#### Instances:
```
/src/NFTEDA/NFTEDA.sol::85 => if (price > maxPrice) {
/src/PaprController.sol::170 => if (request.swapParams.minOut > 0) {
/src/PaprController.sol::172 => } else if (request.debt > 0) {
/src/PaprController.sol::241 => if (amount0Delta > 0) {
/src/PaprController.sol::283 => if (excess > 0) {
/src/PaprController.sol::449 => if (debt > max) {
/src/PaprController.sol::471 => if (newDebt > max) revert IPaprController.ExceedsMaxDebt(newDebt, max);
/src/PaprController.sol::473 => if (newDebt >= 1 << 200) revert IPaprController.DebtAmountExceedsUint200();
/src/PaprController.sol::542 => if (totalOwed > debtCached) {
/src/UniswapOracleFundingRateController.sol::124 => if (_fundingPeriod > 90 days) revert FundingPeriodTooLong();
/src/UniswapOracleFundingRateController.sol::164 => if (targetMarkRatio > targetMarkRatioMax) {
/src/libraries/PoolAddress.sol::23 => if (tokenA > tokenB) (tokenA, tokenB) = (tokenB, tokenA);
```

## [G002] `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables

#### Instances:
```
papr/src/PaprController.sol::326 => info.count -= 1;
papr/src/PaprController.sol::419 => _vaultInfo[account][collateral.addr].count += 1;
```

## [G003] abi.encode() is less efficient than abi.encodePacked()

#### Instances:
```
/src/NFTEDA/NFTEDA.sol::36 => return uint256(keccak256(abi.encode(auction)));
/src/PaprController.sol::197 => abi.encode(msg.sender, collateralAsset, oracleInfo)
/src/PaprController.sol::222 => abi.encode(msg.sender)
/src/PaprController.sol::509 => abi.encode(account, collateralAsset, oracleInfo)
/src/ReservoirOracleUnderwriter.sol::74 => abi.encode(
/src/ReservoirOracleUnderwriter.sol::93 => abi.encode(
/src/libraries/PoolAddress.sol::40 => keccak256(abi.encode(key.token0, key.token1, key.fee)),
```

## [G004] Functions with access control cheaper if payable

If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are `CALLVALUE`(2),`DUP1`(3),`ISZERO`(3),`PUSH2`(3),`JUMPI`(10),`PUSH1`(3),`DUP1`(3),`REVERT`(0),`JUMPDEST`(1),`POP`(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost

#### Instances:
```
/src/PaprController.sol::350 => function setPool(address _pool) external override onlyOwner {
/src/PaprController.sol::355 => function setFundingPeriod(uint256 _fundingPeriod) external override onlyOwner {
/src/PaprController.sol::360 => function setLiquidationsLocked(bool locked) external override onlyOwner {
/src/PaprController.sol::382 => function sendPaprFromAuctionFees(address to, uint256 amount) external override onlyOwner {
/src/PaprController.sol::386 => function burnPaprFromAuctionFees(uint256 amount) external override onlyOwner {
/src/PaprToken.sol::24 => function mint(address to, uint256 amount) external onlyController {
/src/PaprToken.sol::28 => function burn(address account, uint256 amount) external onlyController {
```