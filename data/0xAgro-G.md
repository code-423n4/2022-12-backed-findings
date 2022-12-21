# Gas Report
## Finding Summary
||Issue|Instances|Gas Savings (from provided tests)|
|-|-|-|-|
|[G-01]|`i++`/`i+=1` Used Over `++i`|1|-423
|[G-02]|&& In If Statement(s)|3|-325|
|[G-03]|`i--`/`i-=1` Used Over `--i`|2|-218
|[G-04]|Bit Shift(s) Not Used For MUL/DIV/MOD of Powers of 2|1|-16

### [G-01] `i++`/`i+=1` Used Over `++i`

`++i` increments `i` directly. When using `i++`/`i+=1` solc must create a temporary value, thus resulting in higher gas.

#### Findings:

*/src/PaprController.sol*
Links: [419](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L419).
```solidity
419:	_vaultInfo[account][collateral.addr].count += 1;
```
**Suggested Change**
```solidity
419:	++_vaultInfo[account][collateral.addr].count;
```

### [G-02] && In If Statement(s)

Seperating if statements saves gas.
**Example**
```solidity
//From
if (a != HIGH && b != LOW) {
	//Do Something
}
//To
if (a != HIGH) {
	if (b != LOW) {
		//Do Something
	}
}
```

#### Findings:

*/src/UniswapOracleFundingRateController.sol*
Links: [112](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L112).
```solidity
112:	if (pool != address(0) && !UniswapHelpers.poolsHaveSameTokens(pool, _pool)) revert PoolTokensDoNotMatch();
```

*/src/PaprController.sol*
Links: [290](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L290).
```solidity
290:	if (isLastCollateral && remaining != 0) {
```

### [G-03] `i--`/`i-=1` Used Over `--i`

`--i` increments `i` directly. When using `i--`/`i-=1` solc must create a temporary value, thus resulting in higher gas.

#### Findings:

*/src/PaprController.sol*
Links: [326](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L326).
```solidity
326:	info.count -= 1;
```
**Suggested Change**
```solidity
326:	--info.count
```

*/src/libraries/OracleLibrary.sol*
Links: [48](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/OracleLibrary.sol).
```solidity
326:	twat--;
```
**Suggested Change**
```solidity
326:	--twat;
```

### [G-04] Bit Shift(s) Not Used For MUL/DIV/MOD of Powers of 2

Bitwise operations usually save on gas. Expressions that deal with unsigned integers, a power of 2 and the operations MUL/DIV/MOD can often be re-written with bitwise shifts.

#### Findings:

*/src/libraries/UniswapHelpers.sol*
Links: [111](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/UniswapHelpers.sol#L111).
```solidity
111:	return TickMath.getSqrtRatioAtTick(TickMath.getTickAtSqrtRatio(uint160((token1ONE << 96) / token0ONE)) / 2);
```
**Suggested Change**
```solidity
111:	return TickMath.getSqrtRatioAtTick(TickMath.getTickAtSqrtRatio(uint160((token1ONE << 96) / token0ONE)) >> 1);
```