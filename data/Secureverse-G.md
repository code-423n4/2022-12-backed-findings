### [Gas-01] Multiple address mapping be made as a mapping from a address to a struct 

*2 Instances of this issue*

```solidity
File:   src/PaprController.sol

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L57
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L63
```

### [Gas-02] <x> += <y> consume more gas than <x> = <x> + <y>

*1 Instances of this issue*

```solidity
File:   src/PaprController.sol

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L419
```

### [Gas-03] ```public``` function could be ```external```

*1 Instances of this issue*

```solidity
File:   src/UniswapOracleFundingRateController.sol

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L72
```