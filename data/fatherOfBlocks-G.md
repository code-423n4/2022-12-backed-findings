**src/PaprController.sol**
- L278/537 - the debtCached operation - maxDebtCached can be unchecked since in L276 maxDebtCached is generated and at most it can be equal to debtCached, therefore it will never generate underflow.
The same happens with excess - fee, since fee is a percentage of excess.


**src/libraries/OracleLibrary.sol**
- L48 - Instead of twat-- it is less expensive to do unchecked{--twat;}
