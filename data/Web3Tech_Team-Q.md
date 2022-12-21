
  
    QA-1:
       
	    Description :   using block.timestamp and block.number  leadsto issue if transaction took longer time to end
		            though if the scale of your time-dependent event can vary by 15 seconds and maintain integrity,
		            it is safe to use a block.timestamp.
	   
	    Total No. of Instances Found: 11
	   
	    FileName: src/UniswapOracleFundingRateController.sol
       
	    Line No : 46,55,64,73,100,145,157
	   
	    FileName: src/UniswapOracleFundingRateController.sol
       
	    Line No : 321,325,394
		
		
	    FileName: src/ReservoirOracleUnderwriter.sol.sol
       
	    Line No : 106