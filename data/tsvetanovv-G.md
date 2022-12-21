## [GAS-01] ABI.ENCODE() IS LESS EFFICIENT THAN ABI.ENCODEPACKED()

Use abi.encodePacked() where possible to save gas

PaprController.sol

```
197:        abi.encode(msg.sender, collateralAsset, oracleInfo)
```

## [G-02] `x = x - y` costs less gas than `x -= y`, same for addition

[PaprController.sol](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L326)
[PaprController.sol](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L419)

You can replace all `-=` and `+=` occurrences to save gas