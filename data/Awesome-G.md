# 1. Regular addition/subtraction assignment saves gas

To save gas, use standard addition or subtraction assignment (`x = x + y` or `x = x - y`) rather than the shorthand versions (`x += y` or `x -= y`). This can save approximately 22 gas per occurrence. 

There are two instances of this in the code, located at the following lines:

- [PaprController.sol#L419](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L419)
- [PaprController.sol#L326](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L326)

# 2. Uncheck arithmetics operations that can’t underflow/overflow

Solidity version 0.8+ has an implicit overflow and underflow check on unsigned integers.

When an overflow or an underflow isn’t possible, some gas can be saved using an unchecked block.

There is 1 instance of this issue:

- [PaprController.sol#L419](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L419)

# 3. Using `Storage` Instead of `Memory` for Structs/Arrays Saves Gas

When fetching data from a `storage` location, assigning the data to a `memory` variable causes all fields of the struct/array to be read from `storage`, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new `memory` variable, they incur an additional `MLOAD` rather than a cheap stack read. Instead of declaring the variable with the `memory` keyword, declaring the variable with the `storage` keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incurring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a `memory` variable, is if the full struct/array is being returned by the function, is being passed to a function that requires `memory`, or if the array/struct is being read from another `memory` array/struct.

Affected lines of code:

- [PaprController.sol#L164-L166](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L164-L166)

# 4. Ternary operation is more gas-efficient than `if-else` statement

In some cases, using a ternary operation (`x ? y : z`) can be more efficient than an `if-else` statement in terms of gas usage.

If you find instances where a ternary operation can be used instead of an `if-else` statement, it can save a small amount of gas.

There is 1 instance of this:

- [PaprController.sol#L283-L288](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L283-L288)

# 5. Maximize gas savings in Solidity with `payable` functions

Marking functions as `payable` can be slightly cheaper than using non-`payable` functions, because the Solidity compiler inserts a check into non-`payable` functions that requires `msg.value` to be zero.

This optimization can save a small amount of gas, but it is important to be aware of the security considerations involving Ether held in contracts that it introduces.
More information on this topic can be found in the [Solidity Compiler Discussion](https://github.com/ethereum/solidity/issues/12539).

The affected line of code is located at [PaprController.sol#L350](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L350).
