Using private rather than public for constants and immutable saves gas

If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single increaseDebtAndSell function that returns a tuple of the values of all currently-public constants. Saves 3406-3606 gas in deployment gas due to the compiler not having to create non-payable increaseDebtAndSell functions for deployment calldata, not having to store the bytes of the value outside of where it’s used, and not adding another entry to the method ID table.

File: src/NFTEDA/PaperController.sol;

30 :        uint256 public constant BIPS_ONE = 1e4;


35:    bool public immutable override token0IsUnderlying;

    
 38:   uint256 public immutable override maxLTV;
