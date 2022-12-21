`ReservoirOracleUnderwriter.underwritePriceForCollateral()` repeatedly uses values from the `oracleInfo.message` memory struct.

Each time a value from this struct is used, the bytecode has to calculate correct offsets, then convert from memory to the stack, then use the value.

If these memory struct values are converted to local values at the start of the function, then the memory access and offset calculations are only paid once. Per `forge snapshot`, the following code, plus using these vars in the rest of the function, saves 73 gas each oracle invocation.

```
        ReservoirOracle.Message memory message = oracleInfo.message;
        uint256 oracleTimestamp = message.timestamp;
        bytes32 messageId = message.id;
        bytes memory messagePayload = message.payload;
```

Alternately, just deconstructing the timestamp

```
uint256 oracleTimestamp = oracleInfo.message.timestamp;
```

saves 55 gas, and may look cleaner in the code.