## Summary
### Low Risk Issues List
| Number |Issues Details|Context|
|:--:|:-------|:--:|
|[L-01]|  `twat` value may be truncated| 1 |
|[L-02]| Missing Event for critical parameters change | 1 |
|[L-03]| A single point of failure | 5 |

Total 3 issues


### Non-Critical Issues List
| Number |Issues Details|Context|
|:--:|:-------|:--:|
| [N-01]|Use a single file for all system-wide constants|5|
| [N-02] |NatSpec comments should be increased in contracts ||
| [N-03] |For functions, follow Solidity standard naming conventions| 5 |
| [N-04] |`Function writing` that does not comply with the `Solidity Style Guide`| All contracts |
| [N-05] |Add a timelock to critical functions |4|
| [N-06] |Use a more recent version of Solidity| 10 |
| [N-07] |Insufficient coverage|  |
| [N-08] |Floating pragma | 4 |
| [N-09] |Include return parameters in NatSpec comments|All contracts  |
| [N-10] |Omissions in Events | 4  |
| [N-11] |Need Fuzzing test| 6|
| [N-12] |Add comments for debug to `require`s | 2|

Total 12 issues

### [L-01] `twat` value may be truncated

`delta` and `twapDuration` are int56 , but truncated with `int24`


**Context:**
```js
src/libraries/OracleLibrary.sol:
  33      // adapted from https://github.com/Uniswap/v3-periphery/blob/main/contracts/libraries/OracleLibrary.sol#L30-L40
  34:     function timeWeightedAverageTick(int56 startTick, int56 endTick, int56 twapDuration)
  35:         internal
  36:         view
  37:         returns (int24 twat)
  38:     {
  39:         require(twapDuration != 0, "BP");
  40: 
  41:         unchecked {
  42:             int56 delta = endTick - startTick;
  43: 
  44:             twat = int24(delta / twapDuration);
  45: 
  46:             // Always round to negative infinity
  47:             if (delta < 0 && (delta % (twapDuration) != 0)) {
  48:                 twat--;
  49:             }
  50: 
  51:             twat;
  52:         }
  53:     }

```

Proof of Concept

```solidity
contract Test is DSTest {

    Contract0 c0;
    Contract1 c1;

    function setUp() public {
        c0 = new Contract0();
        c1 = new Contract1();

    }
    function test() public {
        c0.timeWeightedAverageTick(887008,887227000,3);
        c1.timeWeightedAverageTick(887008,887227000,3);
    }
}


contract Contract0{

   function timeWeightedAverageTick(int56 startTick, int56 endTick, int56 twapDuration)
        external
        view
        returns (int24 twat)

    {
        require(twapDuration != 0, "BP");
        unchecked {
            int56 delta = endTick - startTick;
            twat = int24(delta / twapDuration);
            // Always round to negative infinity
            if (delta < 0 && (delta % (twapDuration) != 0)) {
                twat--;
            }
            twat;
        }
    }
}



contract Contract1{

   function timeWeightedAverageTick(int56 startTick, int56 endTick, int56 twapDuration)
        external
        view
        returns (int56 twat)
    {
        require(twapDuration != 0, "BP");
        unchecked {
            int56 delta = endTick - startTick;
            twat = int56(delta / twapDuration);
            // Always round to negative infinity
            if (delta < 0 && (delta % (twapDuration) != 0)) {
                twat--;
            }
            twat;
        }
    }
}


```


```js
Running 1 test for src/test/test.sol:Test
[PASS] test() (gas: 11586)
Traces:

  [11586] Test::test() 
    ├─ [682] Contract0::timeWeightedAverageTick(887008, 887227000, 3) [staticcall]
    │   └─ ← -6543224
    ├─ [682] Contract1::timeWeightedAverageTick(887008, 887227000, 3) [staticcall]
    │   └─ ← 295446664
    └─ ← ()


```

### [L-02] Missing Event for critical parameters change

**Context:**
```js
src/PaprController.sol:
  358  
  359:     /// @inheritdoc IPaprController
  360:     function setLiquidationsLocked(bool locked) external override onlyOwner {
  361:         liquidationsLocked = locked;
  362:     }
```

**Description:**
Events help non-contract tools to track changes, and events prevent users from being surprised by changes

**Recommendation:**
Add Event-Emit

### [L-03]  A single point of failure 

[PaprController.sol#L23](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L23)

## Impact

The `owner` role has a single point of failure and `onlyOwner` can use critical a few functions.

`owner` role in the project:
Owner is not behind a multisig and changes are not behind a timelock.

Even if protocol admins/developers are not malicious there is still a chance for Owner keys to be stolen. In such a case, the attacker can cause serious damage to the project due to important functions. In such a case, users who have invested in project will suffer high financial losses.


`onlyOwner` functions;
```js

5 results - 1 file

src/PaprController.sol:
  349      /// @inheritdoc IPaprController
  350:     function setPool(address _pool) external override onlyOwner {
  351          _setPool(_pool);

  354      /// @inheritdoc IPaprController
  355:     function setFundingPeriod(uint256 _fundingPeriod) external override onlyOwner {

  359      /// @inheritdoc IPaprController
  360:     function setLiquidationsLocked(bool locked) external override onlyOwner {
  361          liquidationsLocked = locked;

  382:     function sendPaprFromAuctionFees(address to, uint256 amount) external override onlyOwner {
  383          papr.safeTransferFrom(address(this), to, amount);

  385  
  386:     function burnPaprFromAuctionFees(uint256 amount) external override onlyOwner {
  387          PaprToken(address(papr)).burn(address(this), amount);

```

This increases the risk of ```A single point of failure```



## Tools Used
Manuel Code Review


## Recommended Mitigation Steps
Add a time lock to  critical functions. Admin-only functions that change critical parameters should emit events and have timelocks. 

Events allow capturing the changed parameters so that off-chain tools/interfaces can register such changes with timelocks that allow users to evaluate them and consider if they would like to engage/exit based on how they perceive the changes as affecting the trustworthiness of the protocol or profitability of the implemented financial services.

Allow only multi-signature wallets to call the function to reduce the likelihood of an attack.

https://twitter.com/danielvf/status/1572963475101556738?s=20&t=V1kvzfJlsx-D2hfnG0OmuQ

Also detail them in documentation and NatSpec comments



### [N-01] Use a single file for all system-wide constants

There are many addresses and constants used in the system. It is recommended to put the most used ones in one file (for example constants.sol, use inheritance to access these values)

  This will help with readability and easier maintenance for future changes. 

constants.sol
Use and import this file in contracts that require access to these values. This is just a suggestion, in some use cases this may result in higher gas usage in the distribution

```solidity
5 results - 4 files

src/PaprController.sol:
  29      /// @dev what 1 = 100% is in basis points (bips)
  30:     uint256 public constant BIPS_ONE = 1e4;
  31  

src/ReservoirOracleUnderwriter.sol:
  34      /// @notice the amount of time to use for the TWAP
  35:     uint256 constant TWAP_SECONDS = 30 days;
  36  
  37      /// @notice the maximum time a given signed oracle message is valid for
  38:     uint256 constant VALID_FOR = 20 minutes;
  39  

src/libraries/PoolAddress.sol:
  7  library PoolAddress {
  8:     bytes32 internal constant POOL_INIT_CODE_HASH = 0xe34f199b19b2b4f47f68442619d555527d244f78a3297ea89325f843f87b8b54;
  9  

src/libraries/UniswapHelpers.sol:
  18  
  19:     IUniswapV3Factory constant FACTORY = IUniswapV3Factory(0x1F98431c8aD98523631AE4a59f267346ea31F984);

```
### [N-02] NatSpec comments should be increased in contracts

**Context:**
```solidity
src/libraries/OracleLibrary.sol:
src/NFTEDA/libraries/EDAPrice.sol:

```

**Description:**
It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation.
In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.
https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

**Recommendation:**
NatSpec comments should be increased in contracts

### [N-03] For functions, follow Solidity standard naming conventions

**Context:**
```solidity

src/libraries/OracleLibrary.sol:

  10:     function getQuoteAtTick(int24 tick, uint128 baseAmount, address baseToken, address quoteToken)
  11:         internal
 
  34:     function timeWeightedAverageTick(int56 startTick, int56 endTick, int56 twapDuration)
  35:         internal
  
  56:     function latestCumulativeTick(address pool) internal view returns (int56) {


src/libraries/PoolAddress.sol:
  22:     function getPoolKey(address tokenA, address tokenB, uint24 fee) internal pure returns (PoolKey memory) {

  31:     function computeAddress(address factory, PoolKey memory key) internal pure returns (address pool) {

```


**Description:**
The above codes don't follow Solidity's standard naming convention,

internal and private functions : the mixedCase format starting with an underscore (_mixedCase starting with an underscore)

### [N-04] `Function writing` that does not comply with the `Solidity Style Guide`

**Context:**
All Contracts

**Description:**
Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

Functions should be grouped according to their visibility and ordered:

 constructor
receive function (if exists)
fallback function (if exists)
external
public
internal
private
within a grouping, place the view and pure functions last

### [N-05] Add a timelock to critical functions

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user).
Consider adding a timelock to:

```solidity
src/PaprController.sol:
  349      /// @inheritdoc IPaprController
  350:     function setPool(address _pool) external override onlyOwner {
  351          _setPool(_pool);

  354      /// @inheritdoc IPaprController
  355:     function setFundingPeriod(uint256 _fundingPeriod) external override onlyOwner {
  356          _setFundingPeriod(_fundingPeriod);

  359      /// @inheritdoc IPaprController
  360:     function setLiquidationsLocked(bool locked) external override onlyOwner {
  361          liquidationsLocked = locked;

  364      /// @inheritdoc IPaprController
  365:     function setAllowedCollateral(IPaprController.CollateralAllowedConfig[] calldata collateralConfigs)
  366          external


```
### [N-06] Use a more recent version of Solidity

**Context:**
```solidity

10 results - 14 files

src/interfaces/IFundingRateController.sol:
  1  // SPDX-License-Identifier: MIT
  2: pragma solidity >=0.8.0;

src/interfaces/IPaprController.sol:
  1  // SPDX-License-Identifier: MIT
  2: pragma solidity >=0.8.0;

src/interfaces/IUniswapOracleFundingRateController.sol:
  1  // SPDX-License-Identifier: MIT
  2: pragma solidity >=0.8.0;

src/libraries/OracleLibrary.sol:
  1  // SPDX-License-Identifier: GPL-2.0-or-later
  2: pragma solidity >=0.8.0;

src/libraries/PoolAddress.sol:
  3  // with updates for solc 8
  4: pragma solidity >=0.8.0;

src/libraries/UniswapHelpers.sol:
  1  // SPDX-License-Identifier: GPL-2.0-or-later
  2: pragma solidity >=0.8.0;

src/NFTEDA/NFTEDA.sol:
  1  // SPDX-License-Identifier: MIT
  2: pragma solidity >=0.8.0;

src/NFTEDA/extensions/NFTEDAStarterIncentive.sol:
  1  // SPDX-License-Identifier: MIT
  2: pragma solidity >=0.8.0;

src/NFTEDA/interfaces/INFTEDA.sol:
  1  // SPDX-License-Identifier: MIT
  2: pragma solidity >=0.8.0;

src/NFTEDA/libraries/EDAPrice.sol:
  1  // SPDX-License-Identifier: MIT
  2: pragma solidity >=0.8.0;

```
**Description:**
For security, it is best practice to use the latest Solidity version.
For the security fix list in the versions;
https://github.com/ethereum/solidity/blob/develop/Changelog.md


**Recommendation:**
Old version of Solidity is used , newer version can be used `(0.8.17)` 

### [N-07] Insufficient coverage

**Description:**
The test coverage rate of the project is 55%. Testing all functions is best practice in terms of security criteria.

```js

| File                                                                                 | % Lines          | % Statements     | % Branches      | % Funcs         |
|--------------------------------------------------------------------------------------|------------------|------------------|-----------------|-----------------|
| src/NFTEDA/NFTEDA.sol                                                                | 95.65% (22/23)   | 96.15% (25/26)   | 87.50% (7/8)    | 100.00% (5/5)   |
| src/NFTEDA/extensions/NFTEDAStarterIncentive.sol                                     | 70.00% (7/10)    | 72.73% (8/11)    | 100.00% (2/2)   | 80.00% (4/5)    |
| src/NFTEDA/libraries/EDAPrice.sol                                                    | 0.00% (0/5)      | 0.00% (0/9)      | 100.00% (0/0)   | 0.00% (0/1)     |
| src/PaprController.sol                                                               | 95.33% (143/150) | 94.92% (168/177) | 80.77% (42/52)  | 100.00% (27/27) |
| src/PaprToken.sol                                                                    | 100.00% (2/2)    | 100.00% (2/2)    | 100.00% (0/0)   | 100.00% (2/2)   |
| src/ReservoirOracleUnderwriter.sol                                                   | 75.00% (9/12)    | 80.00% (12/15)   | 62.50% (5/8)    | 0.00% (0/1)     |
| src/UniswapOracleFundingRateController.sol                                           | 100.00% (49/49)  | 98.31% (58/59)   | 95.45% (21/22)  | 100.00% (12/12) |
| Total                                                                                | 58.57% (263/449) | 58.11% (308/530) | 69.83% (81/116) | 55.20% (69/125) |

```
Due to its capacity, test coverage is expected to be 100%.

### [N-08] Floating pragma

**Description:**
Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.
https://swcregistry.io/docs/SWC-103

Floating Pragma List: 
```solidity
4 results - 4 files

src/PaprController.sol:
  1  //SPDX-License-Identifier: BUSL-1.1
  2: pragma solidity ^0.8.17;
  3  

src/PaprToken.sol:
  1  // SPDX-License-Identifier: MIT
  2: pragma solidity ^0.8.17;
  3  

src/ReservoirOracleUnderwriter.sol:
  1  // SPDX-License-Identifier: MIT
  2: pragma solidity ^0.8.17;
  3  

src/UniswapOracleFundingRateController.sol:
  1  //SPDX-License-Identifier: BUSL-1.1
  2: pragma solidity ^0.8.17;

```


**Recommendation:**
Lock the pragma version and also consider known bugs (https://github.com/ethereum/solidity/releases) for the compiler version that is chosen.

### [N-09] Include return parameters in NatSpec comments

**Context:**
All Contracts

**Description:**
It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation. In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.

https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

**Recommendation:**
Include return parameters in NatSpec comments

_Recommendation  Code Style: (from Uniswap3)_
```js
    /// @notice Adds liquidity for the given recipient/tickLower/tickUpper position
    /// @dev The caller of this method receives a callback in the form of IUniswapV3MintCallback#uniswapV3MintCallback
    /// in which they must pay any token0 or token1 owed for the liquidity. The amount of token0/token1 due depends
    /// on tickLower, tickUpper, the amount of liquidity, and the current price.
    /// @param recipient The address for which the liquidity will be created
    /// @param tickLower The lower tick of the position in which to add liquidity
    /// @param tickUpper The upper tick of the position in which to add liquidity
    /// @param amount The amount of liquidity to mint
    /// @param data Any data that should be passed through to the callback
    /// @return amount0 The amount of token0 that was paid to mint the given amount of liquidity. Matches the value in the callback
    /// @return amount1 The amount of token1 that was paid to mint the given amount of liquidity. Matches the value in the callback
    function mint(
        address recipient,
        int24 tickLower,
        int24 tickUpper,
        uint128 amount,
        bytes calldata data
    ) external returns (uint256 amount0, uint256 amount1);
```

### [N-10] Omissions in Events

Throughout the codebase, events are generally emitted when sensitive changes are made to the contracts. However, some events are missing important parameters

The events should include the new value and old value where possible:

```solidity

src/PaprController.sol:
  349      /// @inheritdoc IPaprController
  350:     function setPool(address _pool) external override onlyOwner {
  351          _setPool(_pool);

  354      /// @inheritdoc IPaprController
  355:     function setFundingPeriod(uint256 _fundingPeriod) external override onlyOwner {
  356          _setFundingPeriod(_fundingPeriod);

  359      /// @inheritdoc IPaprController
  360:     function setLiquidationsLocked(bool locked) external override onlyOwner {
  361          liquidationsLocked = locked;

  364      /// @inheritdoc IPaprController
  365:     function setAllowedCollateral(IPaprController.CollateralAllowedConfig[] calldata collateralConfigs)
  366          external

```
### [N-11] Need Fuzzing test

```solidity
6 results - 2 files

src/PaprController.sol:
  101              collateralArr[i].addr.transferFrom(msg.sender, address(this), collateralArr[i].id);
  102:             unchecked {
  103:                 ++i;
  104              }

  130  
  131:             unchecked {
  132:                 ++i;
  133              }

  374              emit AllowCollateral(collateralConfigs[i].collateral, collateralConfigs[i].allowed);
  375:             unchecked {
  376:                 ++i;
  377              }

  436          uint16 newCount;
  437:         unchecked {
  438:             newCount = _vaultInfo[msg.sender][collateral.addr].count - 1;
  439              _vaultInfo[msg.sender][collateral.addr].count = newCount;

src/libraries/OracleLibrary.sol:
  14      {
  15:         unchecked {
  16:             uint160 sqrtRatioX96 = TickMath.getSqrtRatioAtTick(tick);
  17  

  40  
  41:         unchecked {
  42:             int56 delta = endTick - startTick;


```

**Description:**
In total 2 contracts, 6 unchecked are used, the functions used are critical. For this reason, there must be fuzzing tests in the tests of the project. Not seen in tests.

**Recommendation:**
Use should fuzzing test like Echidna.

As Alberto Cuesta Canada said:
_Fuzzing is not easy, the tools are rough, and the math is hard, but it is worth it. Fuzzing gives me a level of confidence in my smart contracts that I didn’t have before. Relying just on unit testing anymore and poking around in a testnet seems reckless now._

https://medium.com/coinmonks/smart-contract-fuzzing-d9b88e0b0a05

### [N-12] Add comments for debug to `require`s


```solidity
src/PaprController.sol:
  245:             require(amount1Delta > 0); 

src/libraries/PoolAddress.sol:
  32:         require(key.token0 < key.token1);

```
