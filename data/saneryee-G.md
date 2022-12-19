# Gas Optimizations Report

|         | Issue                                                                                                                              | Instances |
| ------- | :--------------------------------------------------------------------------------------------------------------------------------- | :-------: |
| [G-001] | `storage` pointer to a structure is cheaper than copying each value of the structure into `memory`, same for `array` and `mapping` |     2     |
| [G-002] | Functions guaranteed to revert when called by normal users can be marked payable                                                   |     7     |
| [G-003] | Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate                 |     1     |
| [G-004] | Multiple accesses of a `mapping/array` should use a local variable cache                                                           |     1     |
| [G-005] | internal functions only called once can be inlined to save gas                                                                     |     6     |
| [G-006] | Usage of `uints/ints` smaller than 32 bytes (256 bits) incurs overhead                                                             |     1     |
| [G-007] | Replace modifier with function                                                                                                     |     1     |
| [G-008] | Use named returns for local variables where it is possible.                                                                        |     1     |
| [G-009] | revert operator should be in the code as early as reasonably possible                                                              |     1     |

## [G-001] `storage` pointer to a structure is cheaper than copying each value of the structure into `memory`, same for `array` and `mapping`

### Impact

It may not be obvious, but every time you copy a storage `struct/array/mapping` to a `memory` variable, you are literally copying each member by reading it from `storage`, which is expensive. And when you use the `storage` keyword, you are just storing a pointer to the storage, which is much cheaper.

### Findings

Total:2

[src/libraries/OracleLibrary.sol#L57](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6//src/libraries/OracleLibrary.sol#L57)

```solidity
57:    uint32[] memory secondAgos = new uint32[](1);
```

[src/libraries/OracleLibrary.sol#L59](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6//src/libraries/OracleLibrary.sol#L59)

```solidity
59:    (int56[] memory tickCumulatives,) = IUniswapV3Pool(pool).observe(secondAgos);
```

## [G-002] Functions guaranteed to revert when called by normal users can be marked payable

### Impact

If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.
The extra opcodes avoided are `CALLVALUE(2)`,`DUP1(3)`,`ISZERO(3)`,`PUSH2(3)`,`JUMPI(10)`,`PUSH1(3)`,`DUP1(3)`,`REVERT(0)`,`JUMPDEST(1)`,`POP(2)`, which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

### Findings

Total:7

[src/PaprToken.sol#L28](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6//src/PaprToken.sol#L28)

```solidity
28:    function burn(address account, uint256 amount) external onlyController {
```

[src/PaprToken.sol#L24](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6//src/PaprToken.sol#L24)

```solidity
24:    function mint(address to, uint256 amount) external onlyController {
```

[src/PaprController.sol#L382](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6//src/PaprController.sol#L382)

```solidity
382:    function sendPaprFromAuctionFees(address to, uint256 amount) external override onlyOwner {
```

[src/PaprController.sol#L386](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6//src/PaprController.sol#L386)

```solidity
386:    function burnPaprFromAuctionFees(uint256 amount) external override onlyOwner {
```

[src/PaprController.sol#L355](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6//src/PaprController.sol#L355)

```solidity
355:    function setFundingPeriod(uint256 _fundingPeriod) external override onlyOwner {
```

[src/PaprController.sol#L360](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6//src/PaprController.sol#L360)

```solidity
360:    function setLiquidationsLocked(bool locked) external override onlyOwner {
```

[src/PaprController.sol#L350](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6//src/PaprController.sol#L350)

```solidity
350:    function setPool(address _pool) external override onlyOwner {
```

### Recommendation

Mark the function as payable.

## [G-003] Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate

### Impact

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to [not having to recalculate the key's keccak256 hash](https://gist.github.com/IllIllI000/ec23a57daa30a8f8ca8b9681c8ccefb0) (Gkeccak256 - 30 gas) and that calculation's associated stack operations.

### Findings

Total:1

[src/PaprController.sol#L57-63](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6//src/PaprController.sol#L57-63)

```solidity
57:        mapping(ERC721 => mapping(uint256 => address)) public override collateralOwner;
58:
59:        /// @inheritdoc IPaprController
60:        mapping(address => bool) public override isAllowed;
61：
62:        /// @dev account => asset => vaultInfo
63:        mapping(address => mapping(ERC721 => IPaprController.VaultInfo)) private _vaultInfo;
```

### Recommendation

Same as description

## [G-004] Multiple accesses of a `mapping/array` should use a local variable cache

### Impact

The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping's value in a local storage or calldata variable when the value is accessed multiple times, saves ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations. Caching an array's struct avoids recalculating the array offsets into `memory/calldata`

### Findings

Total:1

[src/NFTEDA/extensions/NFTEDAStarterIncentive.sol#L44](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6//src/NFTEDA/extensions/NFTEDAStarterIncentive.sol#L44)

```solidity
44:    auctionState[id] = AuctionState({startTime: uint96(block.timestamp), starter: msg.sender});
```

### Recommendation

Same as description

## [G-005] internal functions only called once can be inlined to save gas

### Impact

Not inlining costs 20 to 40 gas because of two extra `JUMP` instructions and additional stack operations needed for function calls.

### Findings

Total:6

[src/UniswapOracleFundingRateController.sol#L156](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6//src/UniswapOracleFundingRateController.sol#L156)

```solidity
156:    function _multiplier(uint256 _mark_, uint256 cachedTarget) internal view returns (uint256) {
```

[src/UniswapOracleFundingRateController.sol#L122](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6//src/UniswapOracleFundingRateController.sol#L122)

```solidity
122:    function _setFundingPeriod(uint256 _fundingPeriod) internal {
```

[src/UniswapOracleFundingRateController.sol#L111](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6//src/UniswapOracleFundingRateController.sol#L111)

```solidity
111:    function _setPool(address _pool) internal {
```

[src/NFTEDA/NFTEDA.sol#L106](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6//src/NFTEDA/NFTEDA.sol#L106)

```solidity
106:    function _clearAuctionState(uint256 id) internal virtual;
```

[src/NFTEDA/NFTEDA.sol#L101](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6//src/NFTEDA/NFTEDA.sol#L101)

```solidity
101:    function _setAuctionStartTime(uint256 id) internal virtual;
```

[src/NFTEDA/extensions/NFTEDAStarterIncentive.sol#L72](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6//src/NFTEDA/extensions/NFTEDAStarterIncentive.sol#L72)

```solidity
72:    function _setCreatorDiscount(uint256 _auctionCreatorDiscountPercentWad) internal virtual {
```

### Recommendation

Same as description

## [G-006] Usage of `uints/ints` smaller than 32 bytes (256 bits) incurs overhead

### Impact

> When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

[https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html](https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html) Each operation involving a uint8 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed

### Findings

Total:1

[src/libraries/OracleLibrary.sol#L42](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6//src/libraries/OracleLibrary.sol#L42)

```solidity
42:    int56 delta = endTick - startTick;
```

### Recommendation

Same as description

## [G-007] Replace modifier with function

### Impact

Modifiers make code more elegant, but cost more than normal functions.

### Findings

Total:1

[src/PaprToken.sol#L11](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6//src/PaprToken.sol#L11)

```solidity
11:    modifier onlyController() {
```

### Recommendation

Same as description

## [G-008] Use named returns for local variables where it is possible.

### Impact

It is not necessary to have both a named return and a return statement.

### Findings

Total:1

[src/NFTEDA/extensions/NFTEDAStarterIncentive.sol#L60](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6//src/NFTEDA/extensions/NFTEDAStarterIncentive.sol#L60)

```solidity
60:    uint256 price = super._auctionCurrentPrice(id, startTime, auction);
```

## [G-009] revert operator should be in the code as early as reasonably possible

### Impact

`revert` operator should be in the code as early as reasonably possible

### Findings

Total:1

[src/PaprController.sol#L321-322](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6//src/PaprController.sol#L321-322)

```solidity
321:    if (block.timestamp - info.latestAuctionStartTime < liquidationAuctionMinSpacing) {
322:                revert IPaprController.MinAuctionSpacing();
```
