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