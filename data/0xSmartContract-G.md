### Gas Optimizations List
| Number | Optimization Details | Context |
|:--:|:-------| :-----:|
| [G-01] | Add `_checkController()` function to `modifier onlyController()`| 1 |
| [G-02] |No need for type conversion as `pool` is already an address | 1 |
| [G-03] |Before you deploy your contract, activate the optimizer |  |
| [G-04] | Use nested if and, avoid multiple check combinations  | 3 |
| [G-05] | Use ``assembly`` to write _address storage values_  | 5|
| [G-06] | Setting the constructor to `payable`| 5|
| [G-07] | Functions guaranteed to revert_ when callled by normal users can be marked `payable`   | 5 |
| [G-08] | Use abi.encodePacked() instead of abi.encode()| 4 |
| [G-09] |`++x` costs less gas compared to `x += 1`  | 1 |
| [G-10] | Optimize names to save gas | All contracts |
| [G-11] | Use a more recent version of solidity |10|
| [G-12] | The solady Library's Ownable contract is significantly gas-optimized, which can be used  | |

Total 10 issues

### [G-01] Add `_checkController()` function to `modifier onlyController()`

Modifier code is copied in all instances where it's used, increasing bytecode size. By doing a refractor to the internal function, one can reduce bytecode size significantly at the cost of one JUMP. 

https://github.com/PolymathNetwork/polymath-core/pull/548/commits/2dc0286f4e96241eed9603534607431a8a84ba35#diff-8b6746c2c4e7c9e3fca67d62718d70e8


```solidity
src/PaprToken.sol:
  11:     modifier onlyController() {
  12         if (msg.sender != controller) {
  13             revert ControllerOnly();
  14         }
  15         _;
  16     }

```

**Recommendation Code:**

```diff

src/PaprToken.sol:
11:     modifier onlyController() {
+              _checkController();
- 12         if (msg.sender != controller) {
- 13             revert ControllerOnly();
- 14         }
15         _;
16     }


+ function _checkController() internal view {
+ 12      if (msg.sender != controller) {
+ 13         revert ControllerOnly();
+ 14    }
+  }

```
### [G-02] No need for type conversion as `pool` is already an address

```diff
src/libraries/UniswapHelpers.sol:
  69     function deployAndInitPool(address tokenA, address tokenB, uint24 feeTier, uint160 sqrtRatio)
  70         internal
  71         returns (address)
  72     {
  73         IUniswapV3Pool pool = IUniswapV3Pool(FACTORY.createPool(tokenA, tokenB, feeTier));
  74         pool.initialize(sqrtRatio);
  75 
- 76:         return address(pool);
+ 76:         return pool;
  77     }

```
### [G-03] Before you deploy your contract, activate the optimizer

```js
foundry.toml:
   1: [profile.default]
   2: src = 'src'
   3: out = 'out'
   4: libs = ['lib']
   5: solc = "0.8.17"
   6: remappings = [
   7:     '@openzeppelin/=lib/openzeppelin-contracts/',
   8:     '@uniswap/v3-core/=lib/v3-core/',
   9:     '@uniswap/v3-periphery/=lib/v3-periphery/',
  10:     '@reservoir/=lib/oracle/contracts'
  11: ]
  12: fs_permissions = [{ access = "read", path = "./"}]

```

For optimizing gas costs, always use Solidity optimizer. It’s a good practice to set the optimizer as high as possible just until it no longer helps reducing gas costs on function calls. This can be recommended because function calls are intended to be executed way more times than the deployment of the contract, which happens just once.
But, if the project you’re working on is really sensitive about deployment cost, playing with low optimizer values just until it no longer helps reducing the deployment cost should help.

https://docs.soliditylang.org/en/v0.5.4/using-the-compiler.html#using-the-commandline-compiler

### [G-04] Use nested if and, avoid multiple check combinations

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

```solidity
3 results - 3 files

src\PaprController.sol:
  290:         if (isLastCollateral && remaining != 0) {

src\UniswapOracleFundingRateController.sol:
  112:         if (pool != address(0) && !UniswapHelpers.poolsHaveSameTokens(pool, _pool)) revert PoolTokensDoNotMatch();

src\libraries\OracleLibrary.sol:
  47:             if (delta < 0 && (delta % (twapDuration) != 0)) {

```

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L290
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L112
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/OracleLibrary.sol#L47


**Recomendation Code:**

```diff
src\PaprController.sol#L290

- if (isLastCollateral && remaining != 0) {
+ if (isLastCollateral) {
+ if (remaining != 0) {

```

### [G-05] Use ``assembly`` to write _address storage values_ 

```solidity
5 results - 2 files

src\ReservoirOracleUnderwriter.sol:
  51     constructor(address _oracleSigner, address _quoteCurrency) {
  52:         oracleSigner = _oracleSigner;
  53:         quoteCurrency = _quoteCurrency;

src\UniswapOracleFundingRateController.sol:
  34     constructor(ERC20 _underlying, ERC20 _papr, uint256 _targetMarkRatioMax, uint256 _targetMarkRatioMin) {
  35:         underlying = _underlying;
  36:         papr = _papr;

  111     function _setPool(address _pool) internal {
  115:         pool = _pool;
```
https://github.com/code-423n4/2022-12-Stealth-Project/blob/main/maverick-v1/contracts/models/Factory.sol#L30
https://github.com/code-423n4/2022-12-Stealth-Project/blob/main/maverick-v1/contracts/models/Factory.sol#L42
https://github.com/code-423n4/2022-12-Stealth-Project/blob/main/maverick-v1/contracts/models/Pool.sol#L78-L79
https://github.com/code-423n4/2022-12-Stealth-Project/blob/main/maverick-v1/contracts/models/Position.sol#L19
https://github.com/code-423n4/2022-12-Stealth-Project/blob/main/router-v1/contracts/Router.sol#L49-L51

**Recommendation Code:**
```solidity

contracts\PairsContract.sol:

    function _setPool(address _pool) internal {
        assembly {
            sstore(pool.slot, _pool)
         }  
     }
```

### [G-06] Setting the constructor to `payable`

You can cut out 10 opcodes in the creation-time EVM bytecode if you declare a constructor payable. Making the constructor payable eliminates the need for an initial check of ```msg.value == 0``` and saves ```13 gas``` on deployment with no security risks.

**Context:**
```solidity
5 results - 5 files

src/PaprController.sol:
  67:     constructor(

src/PaprToken.sol:
  18:     constructor(string memory name, string memory symbol)

src/ReservoirOracleUnderwriter.sol:
  51:     constructor(address _oracleSigner, address _quoteCurrency) {

src/UniswapOracleFundingRateController.sol:
  34:     constructor(ERC20 _underlying, ERC20 _papr, uint256 _targetMarkRatioMax, uint256 _targetMarkRatioMin) {

src/NFTEDA/extensions/NFTEDAStarterIncentive.sol:
  33:     constructor(uint256 _auctionCreatorDiscountPercentWad) {

```

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L67
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprToken.sol#L18
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/ReservoirOracleUnderwriter.sol#L51
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L34
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol#L33



**Recommendation:**
Set the constructor to ```payable```

### [G-07]  Functions guaranteed to revert_ when callled by normal users can be marked `payable` 

If a function modifier or require such as onlyOwner-admin is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

```solidity
5 results - 1 file

src\PaprController.sol:
  350:     function setPool(address _pool) external override onlyOwner {
  355:     function setFundingPeriod(uint256 _fundingPeriod) external override onlyOwner {
  360:     function setLiquidationsLocked(bool locked) external override onlyOwner {
  382:     function sendPaprFromAuctionFees(address to, uint256 amount) external override onlyOwner {
  386:     function burnPaprFromAuctionFees(uint256 amount) external override onlyOwner {

```

**Recommendation:**
Functions guaranteed to revert when called by normal users can be marked payable  (for only ```onlyOwner``` functions)

### [G-08] Use abi.encodePacked() instead of abi.encode()

```solidity
4 results - 3 files

src/PaprController.sol:
  197:   abi.encode(msg.sender, collateralAsset, oracleInfo)
  509:   abi.encode(account, collateralAsset, oracleInfo)

src/libraries/PoolAddress.sol:
  40:    keccak256(abi.encode(key.token0, key.token1, key.fee)),

src/NFTEDA/NFTEDA.sol:
  36:    return uint256(keccak256(abi.encode(auction)));

```

**Recommendation Code:**
```diff

src/PaprController.sol:
- 197:   abi.encode(msg.sender, collateralAsset, oracleInfo)
+ 197:   abi.encodePacked(msg.sender, collateralAsset, oracleInfo)

```
### [G-09] `++x` costs less gas compared to `x += 1`

```diff

1 result - 1 file

src\PaprController.sol:

  413:     function _addCollateralToVault(address account, IPaprController.Collateral memory collateral) internal { 
- 419:         _vaultInfo[account][collateral.addr].count += 1;
+ 419:         ++_vaultInfo[account][collateral.addr].count;

  325          info.latestAuctionStartTime = uint40(block.timestamp);
- 326:         info.count -= 1;
+ 326:        --info.count;

```

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L419
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L326
### [G-10] Optimize names to save gas

Contracts most called functions could simply save gas by function ordering via ```Method ID```. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because ```22 gas``` are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions. 

**Context:** 
All Contracts


**Recommendation:** 
Find a lower ```method ID``` name for the most called functions for example Call() vs. Call1() is cheaper by ```22 gas```
For example, the function IDs in the ``` PaprContrroller.sol ``` contract will be the most used; A lower method ID may be given.

**Proof of Consept:**
https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92

PaprControllerl.sol function names can be named and sorted according to METHOD ID

```js

Sighash   |   Function Signature
========================
c06ef4b5  =>  addCollateral(IPaprController.Collateral[])
dcf8d683  =>  removeCollateral(address,IPaprController.Collateral[],ReservoirOracleUnderwriter.OracleInfo)
32fcb12a  =>  increaseDebt(address,ERC721,uint256,ReservoirOracleUnderwriter.OracleInfo)
242910be  =>  reduceDebt(address,ERC721,uint256)
150b7a02  =>  onERC721Received(address,address,uint256,bytes)
204d3ac0  =>  increaseDebtAndSell(address,ERC721,IPaprController.SwapParams,ReservoirOracleUnderwriter.OracleInfo)
4d8da1cf  =>  buyAndReduceDebt(address,ERC721,IPaprController.SwapParams)
fa461e33  =>  uniswapV3SwapCallback(int256,int256,bytes)
5e0925a4  =>  purchaseLiquidationAuctionNFT(Auction,uint256,address,ReservoirOracleUnderwriter.OracleInfo)
53e2a6de  =>  startLiquidationAuction(address,IPaprController.Collateral,ReservoirOracleUnderwriter.OracleInfo)
4437152a  =>  setPool(address)
2a5aa292  =>  setFundingPeriod(uint256)
ccc77541  =>  setLiquidationsLocked(bool)
82f3bebf  =>  setAllowedCollateral(IPaprController.CollateralAllowedConfig[])
a6acf125  =>  sendPaprFromAuctionFees(address,uint256)
93d7cde6  =>  burnPaprFromAuctionFees(uint256)
115042d3  =>  maxDebt(uint256)
a8559e32  =>  vaultInfo(address,ERC721)
89fbd332  =>  _addCollateralToVault(address,IPaprController.Collateral)
83d294b7  =>  _removeCollateral(address,IPaprController.Collateral,uint256,uint256)
b6d4cdbe  =>  _increaseDebt(address,ERC721,address,uint256,ReservoirOracleUnderwriter.OracleInfo)
eadaa21b  =>  _reduceDebt(address,ERC721,address,uint256)
99c581aa  =>  _reduceDebtWithoutBurn(address,ERC721,uint256)
0e27a944  =>  _increaseDebtAndSell(address,address,ERC721,IPaprController.SwapParams,ReservoirOracleUnderwriter.OracleInfo)
de052790  =>  _purchaseNFTAndUpdateVaultIfNeeded(Auction,uint256,address)
d4875cc4  =>  _handleExcess(uint256,uint256,uint256,Auction)
54aa2190  =>  _maxDebt(uint256,uint256)

```
### [G-11] Use a more recent version of solidity

```solidity
10 results - 10 files

src/interfaces/IFundingRateController.sol:
  2: pragma solidity >=0.8.0;

src/interfaces/IPaprController.sol:
  2: pragma solidity >=0.8.0;

src/interfaces/IUniswapOracleFundingRateController.sol:
  2: pragma solidity >=0.8.0;

src/libraries/OracleLibrary.sol:
  2: pragma solidity >=0.8.0;

src/libraries/PoolAddress.sol:
  4: pragma solidity >=0.8.0;

src/libraries/UniswapHelpers.sol:
  2: pragma solidity >=0.8.0;

src/NFTEDA/NFTEDA.sol:
  2: pragma solidity >=0.8.0;

src/NFTEDA/extensions/NFTEDAStarterIncentive.sol:
  2: pragma solidity >=0.8.0;

src/NFTEDA/interfaces/INFTEDA.sol:
  2: pragma solidity >=0.8.0;

src/NFTEDA/libraries/EDAPrice.sol:
  2: pragma solidity >=0.8.0;

```

Use a solidity version of at least 0.8.2 to get simple compiler automatic inlining Use a solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads Use a solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than revert()/require() strings.

Solidity 0.8.10 has a useful change that reduced gas costs of external calls which expect a return value. 

In 0.8.15 the conditions necessary for inlining are relaxed. Benchmarks show that the change significantly decreases the bytecode size (which impacts the deployment cost) while the effect on the runtime gas usage is smaller.

In 0.8.17 prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call; Simplify the starting offset of zero-length operations to zero. More efficient overflow checks for multiplication.

### [G-12] The solady Library's Ownable contract is significantly gas-optimized, which can be used

The project uses the ` onlyOwner ` authorization model I recommend using _Solady's highly gas optimized contract._

https://github.com/Vectorized/solady/blob/main/src/auth/OwnableRoles.sol

```solidity

src\PaprController.sol:

  350:     function setPool(address _pool) external override onlyOwner {
 
  355:     function setFundingPeriod(uint256 _fundingPeriod) external override onlyOwner {
  
  360:     function setLiquidationsLocked(bool locked) external override onlyOwner {
 
  365     function setAllowedCollateral(IPaprController.CollateralAllowedConfig[] calldata collateralConfigs)
  368:         onlyOwner
 
  382:     function sendPaprFromAuctionFees(address to, uint256 amount) external override onlyOwner {
  
  386:     function burnPaprFromAuctionFees(uint256 amount) external override onlyOwner {

```

