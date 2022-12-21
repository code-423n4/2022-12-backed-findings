## `abi.encodePacked()` should not be used with dynamic types when passing the result to a hash function such as `keccak256()`
**summary:** It's worth noting that while `abi.encode()` is generally a better choice for hashing dynamic types, it is less efficient in terms of gas consumption compared to `abi.encodePacked()`. This is because `abi.encode()` includes more information (e.g. the length of dynamic types) in the encoded byte array, which requires additional computation by the EVM.

Therefore, it is important to consider both the correctness and efficiency of the encoding method when choosing between `abi.encodePacked()` and `abi.encode()` in Solidity. In general, it is recommended to use `abi.encode()` when passing dynamic types to a hash function, and to use `abi.encodePacked()` for other purposes where efficiency is more important.
```solidity
src/libraries/PoolAddress.sol-33-        pool = address(
src/libraries/PoolAddress.sol-34-            uint160(
src/libraries/PoolAddress.sol-35-                uint256(
src/libraries/PoolAddress.sol-36-                    keccak256(
src/libraries/PoolAddress.sol:37:                        abi.encodePacked(
src/libraries/PoolAddress.sol-38-                            hex"ff",
src/libraries/PoolAddress.sol-39-                            factory,
src/libraries/PoolAddress.sol-40-                            keccak256(abi.encode(key.token0, key.token1, key.fee)),
src/libraries/PoolAddress.sol-41-                            POOL_INIT_CODE_HASH
--
src/ReservoirOracleUnderwriter.sol-66-        returns (uint256)
src/ReservoirOracleUnderwriter.sol-67-    {
src/ReservoirOracleUnderwriter.sol-68-        address signerAddress = ecrecover(
src/ReservoirOracleUnderwriter.sol-69-            keccak256(
src/ReservoirOracleUnderwriter.sol:70:                abi.encodePacked(
src/ReservoirOracleUnderwriter.sol-71-                    "\x19Ethereum Signed Message:\n32",
src/ReservoirOracleUnderwriter.sol-72-                    // EIP-712 structured-data hash
src/ReservoirOracleUnderwriter.sol-73-                    keccak256(
src/ReservoirOracleUnderwriter.sol-74-                        abi.encode(
```
**Mitigation**: To fix the issue of using `abi.encodePacked()` with dynamic types when passing the result to a hash function such as `keccak256()`, you can use the `abi.encode()` function instead of `abi.encodePacked()`.

The `abi.encode()` function includes the length of dynamic types in the encoded byte array, which ensures that the hash function can properly include all of the data in the hash.

## No `indexed` event parameters
**Summary**: Events with no indexed parameters. This could make it harder and inefficient for off-chain tools to analyse them.
**Details**: Indexed parameters (“topics”) are searchable event parameters. They are stored separately from unindexed event parameters in an efficient manner to allow for faster access. This is useful for efficient off-chain-analysis, but it is also more costly gas-wise.
```solidity
src/NFTEDA/extensions/NFTEDAStarterIncentive.sol:21:    event SetAuctionCreatorDiscount(uint256 discount);
src/NFTEDA/interfaces/INFTEDA.sol:37:    event StartAuction(
src/interfaces/IFundingRateController.sol:9:    event UpdateTarget(uint256 newTarget);
src/interfaces/IFundingRateController.sol:11:    event SetFundingPeriod(uint256 fundingPeriod);
```