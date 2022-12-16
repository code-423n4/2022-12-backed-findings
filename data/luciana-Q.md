## Missing `address(0)`checks

 `_oracleSigner` and  `_quoteCurrency` variables in  could be set as `address(0)` as mistake that could NOT be fixed later.

[`ReservoirOracleUnderwriter.sol#L51-L54`](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/ReservoirOracleUnderwriter.sol#L51-L54)

The constructor is presented below:

```solidity
    constructor(address _oracleSigner, address _quoteCurrency) {
        oracleSigner = _oracleSigner;
        quoteCurrency = _quoteCurrency;
    }
```

*Recommendation:* Consider adding `address(0)` check for `_oracleSigner` and  `_quoteCurrency` variables.