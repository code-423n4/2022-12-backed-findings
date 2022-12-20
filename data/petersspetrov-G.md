[G-01]Cache array length outside of loop
If not cached, the solidity compiler will always read the length of the array during each iteration.
 That is, if it is a storage array, this is an extra sload operation 
 (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an 
 extra mload operation (3 additional gas for each iteration except for the first).

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L99
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L118
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L370


[G-02] ++i costs less gas than i++, especially when it's used in for-loops (--i/i-- too)
Saves 5 gas per loop
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L103
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L132
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L376


[G-03] Don't initialize variables with default value
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L118
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L370
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L99


[G-04]x = x - y; costs less gas than x -= y;, same for gathering You can replace all -= and += encounters
 to save gas
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L326
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L419

[G-05] Use calldata instead of memory for function arguments that do not get mutated
Mark data types as calldata instead of memory where possible. This makes it so that the data is 
not automatically loaded into memory.
 If the data passed into the function does not need to be changed (like updating values in an array),
 it can be passed in as calldata. The one exception to this is if 
 the argument must later be passed into another function that takes an argument that specifies memory storage.
 https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprToken.sol#L18-L19
 
[G-06] Use != 0 instead of > 0 for unsigned integer comparison
 https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L290
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L283