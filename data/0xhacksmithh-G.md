### [Gas-01] MULTIPLE ADDRESS MAPPINGS CAN BE COMBINED INTO A SINGLE MAPPING OF AN ADDRESS TO A STRUCT, WHERE APPROPRIATE

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations.

*2 Instances of this issue*

```solidity
File:   src/PaprController.sol

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L57
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L63
```

### [Gas-02] <x> += <y> CONSUME MORE GAS THAN <x> = <x> + <y>

*1 Instances of this issue*

```solidity
File:   src/PaprController.sol

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L419
```

### [Gas-03] ```public``` FUNCTION COULD BE ```external```

*6 Instances of this issue*

```solidity
File:   src/UniswapOracleFundingRateController.sol

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L72
```
```solidity
File:   src/NFTEDA/NFTEDA.sol

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/NFTEDA.sol#L24
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/NFTEDA.sol#L35
```
```solidity
File:   src/UniswapOracleFundingRateController.sol

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L45
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L63
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L72
```


### [Gas-04] ABI.ENCODE() IS LESS EFFICIENT THAN ABI.ENCODEPACKED()
*1 Instances of this issue*

```solidity
File:   src/NFTEDA/NFTEDA.sol

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/NFTEDA.sol#L36
```

### [Gas-05] >= COSTS LESS GAS THAN >

*2 Instances of this issue*

```solidity
File:   src/ReservoirOracleUnderwriter.sol

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/ReservoirOracleUnderwriter.sol#L106
```
```solidity
File:   src/UniswapOracleFundingRateController.sol

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L164
```

### [Gas-06] DIVIDE BY 2 SHOULD BE PERFORM USING BIT SHIFTs
*1 Instances of this issue*

```solidity
File:   src/libraries/UniswapHelpers.sol
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/UniswapHelpers.sol#L111
```