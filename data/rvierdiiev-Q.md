## QA-01. Lock can't be extended without additional funds.
### Description
When user creates lock, then it is created for 5 years and user's voting power is decreasing with time. In case if user wants to add more funds to lock, then lock [is automatically extended](https://github.com/code-423n4/2024-03-neobase/blob/main/src/VotingEscrow.sol#L328). 

The problem is that users don't have any means to extend lock without additional funds, they should provide at least 1 wei.

### Recommendation
Allow users to provide 0 amount to extend time of a lock.

## QA-02. Unsafe casting is used
### Description
When user creates lock, then provided value is uint256. Later [it is casted to int128](https://github.com/code-423n4/2024-03-neobase/blob/main/src/VotingEscrow.sol#L303), which means that in case if provided amount is bigger then int128, then it can be truncated.

### Recommendation
Make sure that casting is safe.

## QA-03. LiquidityGauge is not compatible with fee on transfer tokens
### Description
LiquidityGauge contract [doesn't track amount](https://github.com/code-423n4/2024-03-neobase/blob/main/src/LiquidityGauge.sol#L33) of tokens that were received and then means provided by user amount of tokens.

In case if fee on transfer token will be used as underlying token, then smaller amount can be received by contract, which will break accounting.
### Recommendation
Track amount of tokens that were received.