## Private function with embedded modifier reduces contract size
Consider having the logic of a modifier embedded through a private function to reduce contract size if need be. A private visibility that saves more gas on function calls than the internal visibility is adopted because the modifier will only be making this call inside the contract.

For instance, the modifier instance below may be refactored as follows:

[File: PaprToken.sol#L11-L16](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprToken.sol#L11-L16)

```diff
+    function _onlyController() private view {
+        if (msg.sender != controller) {
+            revert ControllerOnly();
+        }
+    }

    modifier onlyController() {
-        if (msg.sender != controller) {
-            revert ControllerOnly();
-        }
+        _onlyController();
        _;
    }
```
## Use of named returns for local variables saves gas
You can have further advantages in term of gas cost by simply using named return values as temporary local variable.

For instance, the code block below may be refactored as follows:

[File: NFTEDAStarterIncentive.sol#L38-L40](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol#L38-L40)

```diff
-    function auctionStartTime(uint256 id) public view virtual override returns (uint256) {
+    function auctionStartTime(uint256 id) public view virtual override returns (uint256 _auctionStartTime) {
-        return auctionState[id].startTime;
+        _auctionStartTime = auctionState[id].startTime;
    }
```
## Division by 2
A division by 2 can be calculated by shifting one to the right since the div opcode uses 5 gas while SHR opcode uses 3 gas. Additionally, Solidity's division operation also includes a division-by-0 prevention by pass using shifting.

Consider using >>1 by having the code instance below refactored as follows:

[File: UniswapHelpers.sol#L111](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/UniswapHelpers.sol#L111)

```diff
-        return TickMath.getSqrtRatioAtTick(TickMath.getTickAtSqrtRatio(uint160((token1ONE << 96) / token0ONE)) / 2);
+        return TickMath.getSqrtRatioAtTick(TickMath.getTickAtSqrtRatio(uint160((token1ONE << 96) / token0ONE)) >> 1);
```
## `||` costs less gas than its equivalent `&&`
Rule of thumb: `(x && y)` is `(!(!x || !y))`

Even with the 10k Optimizer enabled: `||`, OR conditions cost less than their equivalent `&&`, AND conditions.

As an example, the code line instance below may be refactored as follows:

[File: UniswapOracleFundingRateController.sol#L112](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L112)

```diff
-        if (pool != address(0) && !UniswapHelpers.poolsHaveSameTokens(pool, _pool)) revert PoolTokensDoNotMatch();
+        if (!(pool == address(0) || UniswapHelpers.poolsHaveSameTokens(pool, _pool))) revert PoolTokensDoNotMatch();
```
## Use storage instead of memory for structs/arrays
A storage pointer is cheaper since copying a state struct in memory would incur as many SLOADs and MSTOREs as there are slots. In another words, this causes all fields of the struct/array to be read from storage, incurring a Gcoldsload (2100 gas) for each field of the struct/array, and then further incurring an additional MLOAD rather than a cheap stack read. As such, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables will be much cheaper, involving only Gcoldsload for all associated field reads. Read the whole struct/array into a memory variable only when it is being returned by the function, passed into a function that requires memory, or if the array/struct is being read from another memory array/struct.

Here are some of the instances entailed:

[File: PaprController.sol](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol)

```
        IPaprController.OnERC721ReceivedArgs memory request = abi.decode(data, (IPaprController.OnERC721ReceivedArgs));

        IPaprController.Collateral memory collateral = IPaprController.Collateral(ERC721(msg.sender), _id);
```
## Avoid late checks
Checks should be as early as possible to avoid wasting more gas than is needed when a function reverts. 

For instance, the if block below should come before transferring the NFT collateral to `sendTo`:

[File: PaprController.sol#L444-L451](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L444-L451)

 ```diff
-        collateral.addr.safeTransferFrom(address(this), sendTo, collateral.id);

        uint256 debt = _vaultInfo[msg.sender][collateral.addr].debt;
        uint256 max = _maxDebt(oraclePrice * newCount, cachedTarget);

        if (debt > max) {
            revert IPaprController.ExceedsMaxDebt(debt, max);
        }

+        collateral.addr.safeTransferFrom(address(this), sendTo, collateral.id);
```
## Ternary over `if ... else`
Using ternary operator instead of the if else statement saves gas.

For instance, the code block below may be refactored as follows:

[File: PaprController.sol#L283-L288](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L283-L288)

```
 excess > 0
     ? {
         remaining = _handleExcess(excess, neededToSaveVault, debtCached, auction);
     }
     : {
         _reduceDebt(auction.nftOwner, auction.auctionAssetContract, address(this), price);
         remaining = debtCached - price;
     }
```
## Non-strict inequalities are cheaper than strict ones
In the EVM, there is no opcode for non-strict inequalities (>=, <=) and two operations are performed (> + = or < + =).

As an example, consider replacing `>=` with the strict counterpart `>` in the following inequality instance:

[File: PaprController.sol#L473](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L473)

```diff
-        if (newDebt >= 1 << 200) revert IPaprController.DebtAmountExceedsUint200();
+        if (newDebt > 1 << 200 - 1) revert IPaprController.DebtAmountExceedsUint200();
```
## += and -= cost more gas
`+=` and `-=` generally cost 22 more gas than writing out the assigned equation explicitly. The amount of gas wasted can be quite sizable when repeatedly operated in a loop.

For example, the `+=` instance below may be refactored as follows:

[File: PaprController.sol#L419](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L419)

```diff
-        _vaultInfo[account][collateral.addr].count += 1;
+        _vaultInfo[account][collateral.addr].count = _vaultInfo[account][collateral.addr].count + 1;
```
Similarly, the `-=` instance below may be refactored as follows:

[File: PaprController.sol#L326](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L326)

```
-        info.count -= 1;
+        info.count = info.count - 1;
```
## Function order affects gas consumption
The order of function will also have an impact on gas consumption. Because in smart contracts, there is a difference in the order of the functions. Each position will have an extra 22 gas. The order is dependent on method ID. So, if you rename the frequently accessed function to more early method ID, you can save gas cost. Please visit the following site for further information:

https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92

## Activate the optimizer
Before deploying your contract, activate the optimizer when compiling using “solc --optimize --bin sourceFile.sol”. By default, the optimizer will optimize the contract assuming it is called 200 times across its lifetime. If you want the initial contract deployment to be cheaper and the later function executions to be more expensive, set it to “ --optimize-runs=1”. Conversely, if you expect many transactions and do not care for higher deployment cost and output size, set “--optimize-runs” to a high number.

```
module.exports = {
solidity: {
version: "0.8.17",
settings: {
  optimizer: {
    enabled: true,
    runs: 1000,
  },
},
},
};
```
Please visit the following site for further information:

https://docs.soliditylang.org/en/v0.5.4/using-the-compiler.html#using-the-commandline-compiler

Here's one example of instance on opcode comparison that delineates the gas saving mechanism:

```
for !=0 before optimization
PUSH1 0x00
DUP2
EQ
ISZERO
PUSH1 [cont offset]
JUMPI

after optimization
DUP1
PUSH1 [revert offset]
JUMPI
```
Disclaimer: There have been several bugs with security implications related to optimizations. For this reason, Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them. Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past . A high-severity bug in the emscripten -generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG. Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. Please measure the gas savings from optimizations, and carefully weigh them against the possibility of an optimization-related bug. Also, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.

## Payable access control functions costs less gas
Consider marking functions with access control as `payable`. This will save 20 gas on each call by their respective permissible callers for not needing to have the compiler check for `msg.value`.

For instance, the code line below may be refactored as follows:

[File: PaprController.sol#L350](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L350)

```diff
-    function setPool(address _pool) external override onlyOwner {
+    function setPool(address _pool) external payable override onlyOwner {
```
## Unchecked SafeMath saves gas
"Checked" math, which is default in ^0.8.0 is not free. The compiler will add some overflow checks, somehow similar to those implemented by `SafeMath`. While it is reasonable to expect these checks to be less expensive than the current `SafeMath`, one should keep in mind that these checks will increase the cost of "basic math operation" that were not previously covered. This particularly concerns variable increments in for loops. When no arithmetic overflow/underflow is going to happen, `unchecked { ... }` to use the previous wrapping behavior further saves gas.

For instance, the following code block may be refactored as follows:

[File: PaprController.sol#L419](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L419)

```diff
+ unchecked{
        _vaultInfo[account][collateral.addr].count += 1;
+ }
```
