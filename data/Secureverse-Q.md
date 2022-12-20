
### [LOW-01] Two different and Floating Pragma Used

Instead of Floating solidity, try to use Stable and Locked Solidity version

```solidity
Below contracts using ```^0.8.17```

File:   src/PaprController.sol
File:   src/UniswapOracleFundingRateController.sol
File:   src/PaprToken.sol
File:   src/ReservoirOracleUnderwriter.sol
```
```solidity
Below contracts using ```>=0.8.0 version```

File:   src/NFTEDA/extensions/NFTEDAStarterIncentive.sol
src/NFTEDA/NFTEDA.sol
```

### [LOW-02] Instead of use of transfer() for transfering asset try to implement Openzeppelin safeERC20 library

*5 Instances of this issue*

```solidity
File:   src/PaprController.sol

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L202
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L203
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L514
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L515
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L546
```


### [NC-01] Unused Imports

Library imported but never used inside contract file

*1 Instances of this issue*

```solidity
File:   src/PaprController.sol

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L9
```

### [NC-02] Immutable state variable can make as CONSTANT

*5 Instances of this issue*

```solidity
File:   src/PaprController.sol

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L41-L54
```

### [NC-03] Instead of large number try to use scientific notation

It will increase the readability, that lead to less error pone

*1 Instances of this issue*

```solidity
File:   src/PaprController.sol

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L54
```

### [NC-04] Instead using magic number, try to use CONSTANT state variable

*1 Instances of this issue*

```solidity
File:   src/PaprController.sol

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L473
```

### [NC-05] ```internal``` function that called only once can be inlined inside parent function

*2 Instances of this issue*

```solidity
File:   src/UniswapOracleFundingRateController.sol

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L111-L118
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L122-L130
```