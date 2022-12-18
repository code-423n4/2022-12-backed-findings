QA1. https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L67
Needs to check 0 < _maxLTV < BIPS_ONE.


QA2, https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L183
the following zero address and NOT-THIS-contract check is necessary; othersise, fund will be lost forever.
```
if(proceedsTo == address(0) || proceedsTo ==  address(this) revert InValidAddress();
```

QA3: https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L425
the following zero address and NOT-THIS-contract check is necessary; othersise, fund will be lost forever.
```
if(sendTo == address(0) || sendTo ==  address(this)) revert InValidAddress();

```

QA4: https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L365
We need to check there are no duplicate collateral in ``CollateralAllowedConfig[]``, othersise, they might have conflicting ``allowed`` and create ambiguity. 

QA5: https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L225-L227
Wrong calculation of swap fee, it should be based on ``

QA6. https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L514
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L203
Use safeTransferFrom and safeTransfer instead of Transfer and TransferFrom since 1) Some ERC20 might not return a bool valune; 2) some ERC20 might not revert for failure and return a bool value.

QA7: https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L188
WE need to check and make sure ``params.swapFeeBips < BIPS_ONE``.
Same check is needed for https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L213


