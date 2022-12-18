## [G-01] Don't Initialize Variables with Default Value

Uninitialized variables are assigned with the types default value. Explicitly initializing a variable with it's default value costs unnecesary gas.

```
papr/src/PaprController.sol::99 => for (uint256 i = 0; i < collateralArr.length;) {
papr/src/PaprController.sol::118 => for (uint256 i = 0; i < collateralArr.length;) {
papr/src/PaprController.sol::370 => for (uint256 i = 0; i < collateralConfigs.length;) {
```

## [G-02] Cache Array Length Outside of Loop

Caching the array length outside a loop saves reading it on each iteration, as long as the array's length is not changed during the loop.

```
papr/src/PaprController.sol::99 => for (uint256 i = 0; i < collateralArr.length;) {
papr/src/PaprController.sol::118 => for (uint256 i = 0; i < collateralArr.length;) {
papr/src/PaprController.sol::370 => for (uint256 i = 0; i < collateralConfigs.length;) {
```

## [G-03] Using > 0 costs more gas than != 0 when used on a uint in a require() statement

When dealing with unsigned integer types, comparisons with != 0 are cheaper then with > 0. This change saves 6 gas per instance

```
papr/src/PaprController.sol::245 => require(amount1Delta > 0); // swaps entirely within 0-liquidity regions are not supported
```

## [G-04] Use calldata instead of memory

Use calldata instead of memory for function parameters saves gas if the function argument is only read.

```
papr/src/NFTEDA/NFTEDA.sol::35 => function auctionID(INFTEDA.Auction memory auction) public pure virtual returns (uint256) {
papr/src/NFTEDA/NFTEDA.sol::48 => function _startAuction(INFTEDA.Auction memory auction) internal virtual returns (uint256 id) {
```

## [G-05] Using private rather than public for constants, saves gas

If needed, the value can be read from the verified contract source code. Savings are due to the compiler not having to create non-payable getter functions for deployment calldata, and not adding another entry to the method ID table

```
papr/src/PaprController.sol::35 => bool public immutable override token0IsUnderlying;
papr/src/PaprController.sol::38 => uint256 public immutable override maxLTV;
papr/src/PaprController.sol::41 => uint256 public immutable override liquidationAuctionMinSpacing = 2 days;
papr/src/PaprController.sol::44 => uint256 public immutable override perPeriodAuctionDecayWAD = 0.7e18;
papr/src/PaprController.sol::47 => uint256 public immutable override auctionDecayPeriod = 1 days;
papr/src/PaprController.sol::50 => uint256 public immutable override auctionStartPriceMultiplier = 3;
papr/src/PaprController.sol::54 => uint256 public immutable override liquidationPenaltyBips = 1000;
papr/src/ReservoirOracleUnderwriter.sol::41 => address public immutable oracleSigner;
papr/src/ReservoirOracleUnderwriter.sol::44 => address public immutable quoteCurrency;
papr/src/UniswapOracleFundingRateController.sol::17 => ERC20 public immutable underlying;
papr/src/UniswapOracleFundingRateController.sol::19 => ERC20 public immutable papr;
papr/src/UniswapOracleFundingRateController.sol::25 => uint256 public immutable targetMarkRatioMax;
papr/src/UniswapOracleFundingRateController.sol::27 => uint256 public immutable targetMarkRatioMin;
```

## [G-06] Functions guaranteed to revert when called by normal users can be marked payable

If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost

```
papr/src/PaprController.sol::350 => function setPool(address _pool) external override onlyOwner {
papr/src/PaprController.sol::355 => function setFundingPeriod(uint256 _fundingPeriod) external override onlyOwner {
papr/src/PaprController.sol::360 => function setLiquidationsLocked(bool locked) external override onlyOwner {
papr/src/PaprController.sol::382 => function sendPaprFromAuctionFees(address to, uint256 amount) external override onlyOwner {
papr/src/PaprController.sol::386 => function burnPaprFromAuctionFees(uint256 amount) external override onlyOwner {
papr/src/PaprToken.sol::24 => function mint(address to, uint256 amount) external onlyController {
papr/src/PaprToken.sol::28 => function burn(address account, uint256 amount) external onlyController {
```

## [G-07] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

```
papr/src/PaprController.sol::83 => uint160 initSqrtRatio;
papr/src/PaprController.sol::436 => uint16 newCount;
papr/src/ReservoirOracleUnderwriter.sol::23 => uint8 v;
papr/src/UniswapOracleFundingRateController.sol::29 => uint128 internal _target;
```

## [G-08] Using bools for storage incurs overhead

Booleans are more expensive than uint256 or any type that takes up a full word because each write operation emits an extra SLOAD to first read the slot's contents, replace the bits taken up by the boolean, and then write back. This is the compiler's defense against contract upgrades and pointer aliasing, and it cannot be disabled.
Use uint256(1) and uint256(2) for true/false instead

```
papr/src/PaprController.sol::32 => bool public override liquidationsLocked;
papr/src/PaprController.sol::35 => bool public immutable override token0IsUnderlying;
papr/src/PaprController.sol::60 => mapping(address => bool) public override isAllowed;
```

## [G-09] ++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, for example when used in for- and while-loops

The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas per loop

```
papr/src/PaprController.sol::103 => ++i;
papr/src/PaprController.sol::132 => ++i;
papr/src/PaprController.sol::376 => ++i;
```

## [G-10] <x> += <y> costs more gas than <x> = <x> + <y> for state variables

use <x> = <x> + <y> or <x> = <x> - <y> instead to save gas

```
papr/src/PaprController.sol::326 => info.count -= 1;
papr/src/PaprController.sol::419 => _vaultInfo[account][collateral.addr].count += 1;
```

## [G-11] abi.encode() is less efficient than abi.encodePacked()

use abi.encodePacked() where possible to save gas

```
papr/src/NFTEDA/NFTEDA.sol::36 => return uint256(keccak256(abi.encode(auction)));
papr/src/PaprController.sol::197 => abi.encode(msg.sender, collateralAsset, oracleInfo)
papr/src/PaprController.sol::222 => abi.encode(msg.sender)
papr/src/PaprController.sol::509 => abi.encode(account, collateralAsset, oracleInfo)
papr/src/ReservoirOracleUnderwriter.sol::74 => abi.encode(
papr/src/ReservoirOracleUnderwriter.sol::93 => abi.encode(
```

## [G-12] Use a more recent version of solidity

Use a solidity version of at least 0.8.0 to get overflow protection without SafeMath
Use a solidity version of at least 0.8.2 to get compiler automatic inlining
Use a solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads
Use a solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than revert()/require() strings
Use a solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value

```
papr/src/NFTEDA/NFTEDA.sol::2 => pragma solidity >=0.8.0;
papr/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol::2 => pragma solidity >=0.8.0;
```

## [G-13] Public functions not called by the contract should be declared external instead

Contracts are allowed to override their parents' functions and change the visibility from external to public and can save gas by doing so.

```
papr/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol::38 => function auctionStartTime(uint256 id) public view virtual override returns (uint256) {
papr/src/UniswapOracleFundingRateController.sol::45 => function updateTarget() public override returns (uint256 nTarget) {
```

## [G-14] Not using the named return variables when a function returns, wastes deployment gas

It is not necessary to have both a named return and a return statement.

```
papr/src/UniswapOracleFundingRateController.sol::45 => function updateTarget() public override returns (uint256 nTarget) {
```

## [G-15] Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations.

```
papr/src/PaprController.sol::60 => mapping(address => bool) public override isAllowed;
papr/src/PaprController.sol::63 => mapping(address => mapping(ERC721 => IPaprController.VaultInfo)) private _vaultInfo;
```

## [G-16] internal functions only called once can be inlined to save gas

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

```
papr/src/NFTEDA/NFTEDA.sol::48 => function _startAuction(INFTEDA.Auction memory auction) internal virtual returns (uint256 id) {
papr/src/NFTEDA/NFTEDA.sol::101 => function _setAuctionStartTime(uint256 id) internal virtual;
papr/src/NFTEDA/NFTEDA.sol::106 => function _clearAuctionState(uint256 id) internal virtual;
papr/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol::72 => function _setCreatorDiscount(uint256 _auctionCreatorDiscountPercentWad) internal virtual {
papr/src/UniswapOracleFundingRateController.sol::95 => function _init(uint256 target, address _pool) internal {
papr/src/UniswapOracleFundingRateController.sol::111 => function _setPool(address _pool) internal {
papr/src/UniswapOracleFundingRateController.sol::122 => function _setFundingPeriod(uint256 _fundingPeriod) internal {
papr/src/UniswapOracleFundingRateController.sol::156 => function _multiplier(uint256 _mark_, uint256 cachedTarget) internal view returns (uint256) {
```

## [G-17] internal functions not called by the contract should be removed to save deployment gas

If the functions are required by an interface, the contract should inherit from that interface and use the override keyword

```
papr/src/NFTEDA/NFTEDA.sol::48 => function _startAuction(INFTEDA.Auction memory auction) internal virtual returns (uint256 id) {
papr/src/UniswapOracleFundingRateController.sol::95 => function _init(uint256 target, address _pool) internal {
```