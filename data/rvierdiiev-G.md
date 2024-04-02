## G-01. Remove delegation logic from VotingEscrow.
### Description
VotingEscrow contract is inherited from another protocol. That protocol used delegation of votes. But neobase don't use it at all, but still have code to handle delegation and also [has delegated fields](https://github.com/code-423n4/2024-03-neobase/blob/main/src/VotingEscrow.sol#L54-L55) in each user's lock.

If protocol removes that logic, then it will reduce contract size, which will decrease deployment cost and also it will reduce gas costs for users as delegated fields will not be used and will not take storage.


## G-02. Remove useless block
### Description
VotingEscrow._checkpoint function [have such block](https://github.com/code-423n4/2024-03-neobase/blob/main/src/VotingEscrow.sol#L169-L171). This block is useless as later `userPointHistory` [always changed to new point](https://github.com/code-423n4/2024-03-neobase/blob/main/src/VotingEscrow.sol#L176). That block of code can be cleared.