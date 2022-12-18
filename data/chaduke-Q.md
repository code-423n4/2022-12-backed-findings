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