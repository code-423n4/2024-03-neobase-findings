# [L-01] In `LendingLedger`, once a liquidity gauge is assigned to a given market, it can't be updated or removed

## Links to affected code
https://github.com/code-423n4/2024-03-neobase/blob/d6e6127e6763b93c23ee95cdf7622fe950d9ed30/src/LendingLedger.sol#L154-L171

## Description
Due to the way that the `LendingLedger` contract is implemented, there is currently no functionality for changing the liquidity gauges of markets once they are set to them. This can prove to be problematic in the long run, since the protocol team might need to change or remove some of them at some point in the future due to various different reasons, but they won't be able to do so.

## Recommended Mitigation Steps
Add functionality for updating/removing liquidity gauges from markets

# [NC-01] `LiquidityGauge` does not have to extend `ERC20`, since this contract is already extended by `ERC20Burnable`

## Links to affected code
https://github.com/code-423n4/2024-03-neobase/blob/d6e6127e6763b93c23ee95cdf7622fe950d9ed30/src/LiquidityGauge.sol#L12

## Description
`ERC20Burnable` already extends `ERC20`, so there is no need for `LiquidityGauge` to also do so

## Recommended Mitigation Steps
Remove `ERC20` from the `LiquidityGauge` contract declaration


# [NC-02] Redundant `super::_afterTokenTransfer` call in `LiquidityGauge::_afterTokenTransfer`

## Links to affected code
https://github.com/code-423n4/2024-03-neobase/blob/d6e6127e6763b93c23ee95cdf7622fe950d9ed30/src/LiquidityGauge.sol#L51

## Description
The `super::_afterTokenTransfer` call in `LiquidityGauge::_afterTokenTransfer` is redundant, since the `ERC20::_afterTokenTransfer`, which is inherited by `ERC20Burnable`, that is the super contract of `LiquidityGauge` has a empty implementation:
```solidity
    function _afterTokenTransfer(address from, address to, uint256 amount) internal virtual {}
```

## Recommended Mitigation Steps
Remove the `super::_afterTokenTransfer` call from `LiquidityGauge::_afterTokenTransfer`


# [NC-03] `LendingLedger::whiteListLendingMarket` lacks a NatSpec for the `_hasGauge` parameter

## Links to affected code
https://github.com/code-423n4/2024-03-neobase/blob/d6e6127e6763b93c23ee95cdf7622fe950d9ed30/src/LendingLedger.sol#L151-L154

## Description
Unlike all other parameters of the `LendingLedger::whiteListLendingMarket` function, `_hasGauge` lacks a param tag that describes it, which makes it's purpose harder to comprehend at first glance

## Recommended Mitigation Steps
For the sake of consistency and readability, add a NatSpec param tag that describes the `_hasGauge` parameter of `LendingLedger::whiteListLendingMarket`