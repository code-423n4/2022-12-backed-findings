# Gas Optimizations Report

## [G-01] revert operator should be in the code as early as reasonably possible

### Impact

`revert` operator should be in the code as early as reasonably possible

### Findings

Total:1

[src/PaprController.sol#L321-322](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6//src/PaprController.sol#L321-322)

```solidity
321:    if (block.timestamp - info.latestAuctionStartTime < liquidationAuctionMinSpacing) {
322:                revert IPaprController.MinAuctionSpacing();
```

## [G-02] `storage` pointer to a structure is cheaper than copying each value of the structure into `memory`, same for `array` and `mapping`

### Impact

It may not be obvious, but every time you copy a storage `struct/array/mapping` to a `memory` variable, you are literally copying each member by reading it from `storage`, which is expensive. And when you use the `storage` keyword, you are just storing a pointer to the storage, which is much cheaper.

### Findings

Total:1

[src/libraries/OracleLibrary.sol#L57](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6//src/libraries/OracleLibrary.sol#L57)

```solidity
57:    uint32[] memory secondAgos = new uint32[](1);
```

## [G-03] Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate

### Impact

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to [not having to recalculate the key's keccak256 hash](https://gist.github.com/IllIllI000/ec23a57daa30a8f8ca8b9681c8ccefb0) (Gkeccak256 - 30 gas) and that calculation's associated stack operations.

### Findings

Total:1

[src/PaprController.sol#L56-63](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6//src/PaprController.sol#L56-63)

```solidity
56:        /// @inheritdoc IPaprController
57:        mapping(ERC721 => mapping(uint256 => address)) public override collateralOwner;
58:
59:        /// @inheritdoc IPaprController
60:        mapping(address => bool) public override isAllowed;
61：
62:        /// @dev account => asset => vaultInfo
63:        mapping(address => mapping(ERC721 => IPaprController.VaultInfo)) private _vaultInfo;
```

## [G-04] Use named returns for local variables where it is possible.

### Impact

It is not necessary to have both a named return and a return statement.

### Findings

Total:1

[src/NFTEDA/extensions/NFTEDAStarterIncentive.sol#L52-L67](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol#L52-L67)

```solidity
    /// @inheritdoc NFTEDA
    function _auctionCurrentPrice(uint256 id, uint256 startTime, INFTEDA.Auction memory auction)
        internal
        view
        virtual
        override
        returns (uint256)
    {
        uint256 price = super._auctionCurrentPrice(id, startTime, auction);


        if (msg.sender == auctionState[id].starter) {
            price = FixedPointMathLib.mulWadUp(price, _pricePercentAfterDiscount);
        }


        return price;
    }
```