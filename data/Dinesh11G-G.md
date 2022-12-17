### 1st Bug
Use != 0 instead of > 0 for Unsigned Integer Comparison

#### Impact
Issue Information: 
When dealing with unsigned integer types, comparisons with != 0 are cheaper than with > 0.

Example
🤦 Bad:

// `a` being of type unsigned integer
require(a > 0, "!a > 0");
🚀 Good:

// `a` being of type unsigned integer
require(a != 0, "!a > 0");

#### Findings:
```
papr/src/UniswapOracleFundingRateController.sol::171 => // safe to cast because targetMarkRatio > 0
papr/src/interfaces/IPaprController.sol::48 => /// @dev debt is ignored in favor of `swapParams.amount` of minOut > 0
```
#### Tools used
Manual code review

### 2nd Bug
Use immutable for OpenZeppelin AccessControl's Roles Declarations

#### Impact
Issue Information: 
⚡️ Only valid for solidity versions <0.6.12 ⚡️

Access roles marked as constant results in computing the keccak256 operation each time the variable is used because assigned operations for constant variables are re-evaluated every time.

Changing the variables to immutable results in computing the hash only once on deployment, leading to gas savings.

Example
🤦 Bad:

bytes32 public constant GOVERNOR_ROLE = keccak256("GOVERNOR_ROLE");
🚀 Good:

bytes32 public immutable GOVERNOR_ROLE = keccak256("GOVERNOR_ROLE");

#### Findings:
```
papr/src/NFTEDA/NFTEDA.sol::36 => return uint256(keccak256(abi.encode(auction)));
papr/src/ReservoirOracleUnderwriter.sol::69 => keccak256(
papr/src/ReservoirOracleUnderwriter.sol::73 => keccak256(
papr/src/ReservoirOracleUnderwriter.sol::75 => keccak256("Message(bytes32 id,bytes payload,uint256 timestamp)"),
papr/src/ReservoirOracleUnderwriter.sol::77 => keccak256(oracleInfo.message.payload),
papr/src/ReservoirOracleUnderwriter.sol::92 => bytes32 expectedId = keccak256(
papr/src/ReservoirOracleUnderwriter.sol::94 => keccak256("ContractWideCollectionPrice(uint8 kind,uint256 twapSeconds,address contract)"),
papr/src/libraries/PoolAddress.sol::36 => keccak256(
papr/src/libraries/PoolAddress.sol::40 => keccak256(abi.encode(key.token0, key.token1, key.fee)),
```
#### Tools used
Manual code review

### 3rd Bug
Long Revert Strings

#### Impact
Issue Information: 
Shortening revert strings to fit in 32 bytes will decrease gas costs for deployment and gas costs when the revert condition has been met.

If the contract(s) in scope allow using Solidity >=0.8.4, consider using Custom Errors as they are more gas efficient while allowing developers to describe the error in detail using NatSpec.

Example
🤦 Bad:

require(condition, "UniswapV3: The reentrancy guard. A transaction cannot re-enter the pool mid-swap");
🚀 Good (with shorter string):

// TODO: Provide link to a reference of error codes
require(condition, "LOK");
🚀 Good (with custom errors):

/// @notice A transaction cannot re-enter the pool mid-swap.
error NoReentrancy();

// ...

if (!condition) {
    revert NoReentrancy();
}

#### Findings:
```
papr/src/NFTEDA/NFTEDA.sol::6 => import {SafeTransferLib} from "solmate/utils/SafeTransferLib.sol";
papr/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol::4 => import {FixedPointMathLib} from "solmate/utils/FixedPointMathLib.sol";
papr/src/NFTEDA/libraries/EDAPrice.sol::4 => import {FixedPointMathLib} from "solmate/utils/FixedPointMathLib.sol";
papr/src/NFTEDA/libraries/EDAPrice.sol::5 => import {SafeCast} from "v3-core/contracts/libraries/SafeCast.sol";
papr/src/PaprController.sol::6 => import {SafeTransferLib} from "solmate/utils/SafeTransferLib.sol";
papr/src/PaprController.sol::7 => import {FixedPointMathLib} from "solmate/utils/FixedPointMathLib.sol";
papr/src/PaprController.sol::8 => import {INFTEDA, NFTEDAStarterIncentive} from "./NFTEDA/extensions/NFTEDAStarterIncentive.sol";
papr/src/PaprController.sol::9 => import {Ownable2Step} from "openzeppelin-contracts/access/Ownable2Step.sol";
papr/src/PaprController.sol::13 => import {UniswapOracleFundingRateController} from "./UniswapOracleFundingRateController.sol";
papr/src/PaprToken.sol::19 => ERC20(string.concat("papr ", name), string.concat("papr", symbol), 18)
papr/src/ReservoirOracleUnderwriter.sol::75 => keccak256("Message(bytes32 id,bytes payload,uint256 timestamp)"),
papr/src/ReservoirOracleUnderwriter.sol::94 => keccak256("ContractWideCollectionPrice(uint8 kind,uint256 twapSeconds,address contract)"),
papr/src/UniswapOracleFundingRateController.sol::5 => import {FixedPointMathLib} from "solmate/utils/FixedPointMathLib.sol";
papr/src/UniswapOracleFundingRateController.sol::13 => } from "./interfaces/IUniswapOracleFundingRateController.sol";
papr/src/interfaces/IPaprController.sol::4 => import {ReservoirOracleUnderwriter} from "../ReservoirOracleUnderwriter.sol";
papr/src/interfaces/IPaprController.sol::7 => import {INFTEDA} from "../NFTEDA/extensions/NFTEDAStarterIncentive.sol";
papr/src/libraries/OracleLibrary.sol::4 => import {IUniswapV3Pool} from "v3-core/contracts/interfaces/IUniswapV3Pool.sol";
papr/src/libraries/UniswapHelpers.sol::4 => import {IUniswapV3Pool} from "v3-core/contracts/interfaces/IUniswapV3Pool.sol";
papr/src/libraries/UniswapHelpers.sol::5 => import {IUniswapV3Factory} from "v3-core/contracts/interfaces/IUniswapV3Factory.sol";
papr/src/libraries/UniswapHelpers.sol::8 => import {SafeCast} from "v3-core/contracts/libraries/SafeCast.sol";
```
#### Tools used
Manual code review

### 4th Bug
Use Shift Right/Left instead of Division/Multiplication if possible

#### Impact
Issue Information: 
A division/multiplication by any number x being a power of 2 can be calculated by shifting log2(x) to the right/left.

While the DIV opcode uses 5 gas, the SHR opcode only uses 3 gas. Furthermore, Solidity's division operation also includes a division-by-0 prevention which is bypassed using shifting.

Example
🤦 Bad:

uint256 b = a / 2;
uint256 c = a / 4;
uint256 d = a * 8;
🚀 Good:

uint256 b = a >> 1;
uint256 c = a >> 2;
uint256 d = a << 3;

#### Findings:
```
papr/src/libraries/UniswapHelpers.sol::111 => return TickMath.getSqrtRatioAtTick(TickMath.getTickAtSqrtRatio(uint160((token1ONE << 96) / token0ONE)) / 2);
```
#### Tools used
Manual code review