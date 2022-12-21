### No. 1
In the function `_removeCollateral`, it is allowed for `onReceive` hook, so it is possible that during the hook the number of user's collateral change (for example by removing collateral).
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L424
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L444

But after this hook, the variable `newCount` is read which is not holding the last state, it has only the old state value of user's collateral count.
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L447

So, it is better to get the latest user's collateral count again to have the correct value:
```
uint256 max = _maxDebt(oraclePrice * _vaultInfo[msg.sender][collateral.addr].count, cachedTarget);
```