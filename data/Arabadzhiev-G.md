# [G-01]  All delegation-related state variables in the `VotingEscrow` contract are redundant

## Links to affected code
https://github.com/code-423n4/2024-03-neobase/blob/d6e6127e6763b93c23ee95cdf7622fe950d9ed30/src/VotingEscrow.sol#L54-L55

## Description
Since in the current implementation of the `VotingEscrow` the `delegate` function has been removed, all delegation-related state variables can be removed in order to save gas from unnecessary use of storage space and SLOAD/SSTORE operations.

## Recommended Mitigation Steps
Remove all delegation-related state variables and logic

# [G-02] Checking whether `power_used` is greater than or equal 0 is redundant, as it is an `uint` value

## Links to affected code
https://github.com/code-423n4/2024-03-neobase/blob/d6e6127e6763b93c23ee95cdf7622fe950d9ed30/src/GaugeController.sol#L419

## Description
The first check from the following require statement is redundant, since `uint` values can not be less than 0:

```solidity
    require(power_used >= 0 && power_used <= 10_000, "Used too much power");
```

## Recommended Mitigation Steps
Remove the first check from that require statement:

```diff
-   require(power_used >= 0 && power_used <= 10_000, "Used too much power");
+   require(power_used <= 10_000, "Used too much power");
```
