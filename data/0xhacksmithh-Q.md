### [Low-01] ECRECOVER() NOT CHECKED FOR SIGNER ADDRESS OF ZERO

The ecrecover() function returns an address of zero when the signature does not match. This can cause problems if address zero is ever the owner of assets, and someone uses the permit function on address zero. If that happens, any invalid signature will pass the checks, and the assets will be stealable. In this case, the asset of concern is the vault’s ERC20 token, and fortunately OpenZeppelin’s implementation does a good job of making sure that address zero is never able to have a positive balance. If this contract ever changes to another ERC20 implementation that is laxer in its checks in favor of saving gas, this code may become a problem.

*1 Instances of this issue*

```solidity
File:   src/ReservoirOracleUnderwriter.sol

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/ReservoirOracleUnderwriter.sol#L68-L86
```

### [LOW-02] Absence of zero address check for ```oracleSigner``` during contract deployment
*1 Instances of this issue*

```solidity
File:   src/PaprController.sol

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L74
```

### [LOW-03] Two different and Floating Pragma Used

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
File:   src/NFTEDA/NFTEDA.sol
File:   src/NFTEDA/libraries/EDAPrice.sol
File:   src/libraries/PoolAddress.sol
File:   src/libraries/OracleLibrary.sol
File:   src/libraries/UniswapHelpers.sol
File:   src/interfaces/IUniswapOracleFundingRateController.sol
File:   src/interfaces/IFundingRateController.sol
File:   src/NFTEDA/interfaces/INFTEDA.sol
File:   src/interfaces/IPaprController.sol
```

### [LOW-04] Instead of use of transfer() for transfering asset try to implement Openzeppelin safeERC20 library

*5 Instances of this issue*

```solidity
File:   src/PaprController.sol

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L202
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L203
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L514
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L515
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L546
```

### [NC-01] ```internal``` function that called only once can be inlined inside parent function

*2 Instances of this issue*

```solidity
File:   src/UniswapOracleFundingRateController.sol

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L111-L118
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L122-L130
```

### [NC-02] Absence of error message in ```require()``` condition
*1 Instances of this issue*

```solidity
File:   src/libraries/PoolAddress.sol

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/PoolAddress.sol#L32
```

### [NC-03] Should return a named return value rather than an expression
*1 Instances of this issue*

```solidity
File:   src/libraries/UniswapHelpers.sol
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/UniswapHelpers.sol#L94
```


### [NC-04] Unused Imports

Library imported but never used inside contract file

*1 Instances of this issue*

```solidity
File:   src/PaprController.sol

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L9
```

### [NC-05] Immutable state variable can make as CONSTANT

*1 Instances of this issue*

```solidity
File:   src/PaprController.sol

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L41-L54
```

### [NC-06] Instead of large number try to use scientific notation

It will increase the readability, that lead to less error pone

*4 Instances of this issue*

```solidity
File:   src/PaprController.sol

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L54
```
```solidity
File:   src/libraries/OracleLibrary.sol

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/OracleLibrary.sol#L23
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/OracleLibrary.sol#L25
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/OracleLibrary.sol#L28
```

### [NC-07] Instead using magic number, try to use CONSTANT state variable

*1 Instances of this issue*

```solidity
File:   src/PaprController.sol

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L473
```

