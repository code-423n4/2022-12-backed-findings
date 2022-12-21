PaprController.sol ```liquidationsLocked``` only prevents liquidation, it does not prevent borrowing，Suggest adding a pause switch to stop borrowing and avoid security risks
```solidity
contract PaprController is
+    bool paused;


    function _increaseDebt(
        address account,
        ERC721 asset,
        address mintTo,
        uint256 amount,
        ReservoirOracleUnderwriter.OracleInfo memory oracleInfo
    ) internal {
+    require(!paused);
...

+  function setPaused(bool _paused) external onlyOwner{
+      paused = _paused;
+   }    
```