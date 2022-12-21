## Table of Contents
- Defined Variables Used Only Once
- X = X + Y costs less gass than X += Y
- Change Function Visibility Public to External
- Use Bit Shifting Instead of Multiplication/Division of 2
- Using Elements Smaller than 32 bytes (256 bits) Might Use More Gas
- ++i Costs Less Gas than i++

&ensp;
### Defined Variables Used Only Once

#### Issue
Certain variables is defined even though they are used only once.
Remove these unnecessary variables to save gas.
For cases where it will reduce the readability, one can use comments to help describe
what the code is doing.

#### PoC
1. Remove `_pool` variable of constructor() of PaprController.sol 
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L92-L94
Mitigation:
Delete line 92 and replace line 94 like shown below
```solidity
        _init(underlyingONE, UniswapHelpers.deployAndInitPool(address(underlying), address(papr), 10000, initSqrtRatio));
```

2. Remove `hasFee` variable of constructor() of PaprController.sol 
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L213
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L225
Mitigation:
Delete line 213 and replace line 225 like shown below
```solidity
        if (params.swapFeeBips != 0) {
```

3. Remove `payer` variable of uniswapV3SwapCallback() of PaprController.sol 
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L252-L253
M.itigation:
Delete line 252 and replace line 253 like shown below
```solidity
            underlying.safeTransferFrom(abi.decode(_data, (address)), msg.sender, amountToPay);
```

4. Remove `cachedTarget` variable of _increaseDebt() of PaprController.sol 
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L463
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L469
Mitigation:
Delete line 463 and replace line 469 like shown below
```solidity
        uint256 max = _maxDebt(_vaultInfo[account][asset].count * oraclePrice, updateTarget());
```

5. Remove `cachedTarget` variable of _increaseDebt() of PaprController.sol 
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L466
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L469
Mitigation:
Delete line 466 and replace line 469 like shown below
```solidity
        uint256 max = _maxDebt(_vaultInfo[account][asset].count * underwritePriceForCollateral(asset, ReservoirOracleUnderwriter.PriceKind.LOWER, oracleInfo), cachedTarget);
```

6. Remove `credit` variable of _handleExcess() of PaprController.sol 
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L537-L538
Mitigation:
Delete line 537 and replace line 538 like shown below
```solidity
        uint256 totalOwed = (excess - fee) + neededToSaveVault;
```

7. Remove `maxLoanUnderlying` variable of _maxDebt() of PaprController.sol 
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L557-L558
Mitigation:
Delete line 557 and replace line 558 like shown below
```solidity
        return (totalCollateraValue * maxLTV) / cachedTarget;
```

#### Mitigation
Don't define variable that is used only once.
Details are listed on above PoC.

&ensp;
### X = X + Y costs less gas than X += Y

#### Issue
X = X + Y costs less gas than X += Y for state variables.
It saves about 8 gas each instance.

#### PoC
Total of 2 instance found.
```solidity
./PaprController.sol:419:        _vaultInfo[account][collateral.addr].count += 1;
./PaprController.sol:326:        info.count -= 1;
```

&ensp;
### Change Function Visibility Public to External

#### Issue
If the function is not called internally, it is cheaper to set your function visibility to external instead of public.

#### PoC
Total of 2 instances found.

1. NFTEDA.sol:auctionCurrentPrice()
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/NFTEDA.sol#L24

2. UniswapOracleFundingRateController.sol:mark() 
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L72

#### Mitigation
Change the function visibility to external.

&ensp;
### Use Bit Shifting Instead of Multiplication/Division of 2

#### Issue
The MUL and DIV opcodes cost 5 gas but SHL and SHR only costs 3 gas.
Since MUL/DIV and SHL/SHR result the same, use cheaper bit shifting.

#### PoC
Total of 1 instance found.
```solidity
./UniswapHelpers.sol:111:        return TickMath.getSqrtRatioAtTick(TickMath.getTickAtSqrtRatio(uint160((token1ONE << 96) / token0ONE)) / 2);
```

#### Mitigation
Use bit shifting instead of multiplication/division.
Example:
```solidity
uint256 center = upper - (upper - lower) / 2; 
Good:
uint256 center = upper - (upper - lower) >> 2; 
```

&ensp;
### Using Elements Smaller than 32 bytes (256 bits) Might Use More Gas

#### Issue
Since EVM operates on 32 bytes at a time, if the element is smaller than that, the EVM must use more operations
in order to reduce the elements from 32 bytes to specified size. Therefore it is more gas saving to use 32 bytes 
unless it is used in a struct or state variable in order to reduce storage slot. 

Reference: https://docs.soliditylang.org/en/v0.8.15/internals/layout_in_storage.html

#### PoC
Total of 8 instances found.
```
./UniswapHelpers.sol:37:        uint160 sqrtPriceLimitX96,
./UniswapHelpers.sol:69:    function deployAndInitPool(address tokenA, address tokenB, uint24 feeTier, uint160 sqrtRatio)
./UniswapHelpers.sol:111:        return TickMath.getSqrtRatioAtTick(TickMath.getTickAtSqrtRatio(uint160((token1ONE << 96) / token0ONE)) / 2);
./PoolAddress.sol:22:    function getPoolKey(address tokenA, address tokenB, uint24 fee) internal pure returns (PoolKey memory) {
./PaprController.sol:83:        uint160 initSqrtRatio;
./PaprController.sol:436:        uint16 newCount;
./OracleLibrary.sol:10:    function getQuoteAtTick(int24 tick, uint128 baseAmount, address baseToken, address quoteToken)
./OracleLibrary.sol:16:            uint160 sqrtRatioX96 = TickMath.getSqrtRatioAtTick(tick);
```

#### Mitigation
I suggest using uint256 instead of anything smaller and downcast where needed.

&ensp;
### ++i Costs Less Gas than i++

#### Issue
Prefix increments/decrements (++i or --i) costs cheaper gas than 
postfix increment/decrements (i++ or i--).

#### PoC
```solidity
./OracleLibrary.sol:48:                twat--;
```

#### Mitigation
Change it to postfix increments/decrements.
It saves 6 gas per loop.

&ensp;

```
./PoolAddress.sol:32:        require(key.token0 < key.token1);
./PaprController.sol:245:            require(amount1Delta > 0); // swaps entirely within 0-liquidity regions are not supported
```