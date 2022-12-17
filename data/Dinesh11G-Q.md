### 1st Bug
Unsafe ERC20 Operation(s)

#### Impact
Issue Information: 
ERC20 operations can be unsafe due to different implementations and vulnerabilities in the standard.

It is therefore recommended to always either use OpenZeppelin's SafeERC20 library or at least to wrap each operation in a require statement.

To circumvent ERC20's approve functions race-condition vulnerability use OpenZeppelin's SafeERC20 library's safe{Increase|Decrease}Allowance functions.

In case the vulnerability is of no danger for your implementation, provide enough documentation explaining the reasonings.

Example
🤦 Bad:

IERC20(token).transferFrom(msg.sender, address(this), amount);
🚀 Good (using OpenZeppelin's SafeERC20):

import {SafeERC20} from "openzeppelin/token/utils/SafeERC20.sol";

// ...

IERC20(token).safeTransferFrom(msg.sender, address(this), amount);
🚀 Good (using require):

bool success = IERC20(token).transferFrom(msg.sender, address(this), amount);
require(success, "ERC20 transfer failed");

#### Findings:
```
papr/src/PaprController.sol::101 => collateralArr[i].addr.transferFrom(msg.sender, address(this), collateralArr[i].id);
papr/src/PaprController.sol::202 => underlying.transfer(params.swapFeeTo, fee);
papr/src/PaprController.sol::203 => underlying.transfer(proceedsTo, amountOut - fee);
papr/src/PaprController.sol::226 => underlying.transfer(params.swapFeeTo, amountIn * params.swapFeeBips / BIPS_ONE);
papr/src/PaprController.sol::514 => underlying.transfer(params.swapFeeTo, fee);
papr/src/PaprController.sol::515 => underlying.transfer(proceedsTo, amountOut - fee);
papr/src/PaprController.sol::546 => papr.transfer(auction.nftOwner, totalOwed - debtCached);
```
#### Tools used
Manual code review

### 2nd Bug
Unspecific Compiler Version Pragma

#### Impact
Issue Information: 
Avoid floating pragmas for non-library contracts.

While floating pragmas make sense for libraries to allow them to be included with multiple different versions of applications, it may be a security risk for application implementations.

A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain.

It is recommended to pin to a concrete compiler version.

Example
🤦 Bad:

pragma solidity ^0.8.0;
🚀 Good:

pragma solidity 0.8.4;

#### Findings:
```
papr/src/PaprController.sol::2 => pragma solidity ^0.8.17;
papr/src/PaprToken.sol::2 => pragma solidity ^0.8.17;
papr/src/ReservoirOracleUnderwriter.sol::2 => pragma solidity ^0.8.17;
papr/src/UniswapOracleFundingRateController.sol::2 => pragma solidity ^0.8.17;
```
#### Tools used
Manual code review