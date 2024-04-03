- [L01 Escrow lock duration cannot be extended for free](#l01-escrow-lock-duration-cannot-be-extended-for-free)
- [L02 VotingEscrow has unimplemented delegation mechanism](#l02-votingescrow-has-unimplemented-delegation-mechanism)

# L01 Escrow lock duration cannot be extended for free

## Description

The `VotingEscrow` contract allows users to lock tokens in exchange for voting power. Whenever users lock their tokens, their tokens are locked for a period of 5 years by default. They receive voting power in exchange, which gradually decreases over time and hits 0 when the lock period is over.

Users can also lock more tokens via the `increaseAmount` function, which allows them to add more funds and also resets the lock duration back to 5 years.

```solidity
require(_value > 0, "Only non zero amount");
require(msg.value == _value, "Invalid value");
newLocked.end = _floorToWeek(block.timestamp + LOCKTIME);
locked[msg.sender] = newLocked;
```

This is an important function since this allows users to constantly keep their lock maxed out, giving them the largest amount of voting power. The issue is that users cannot extend their lock for free.

Due to the `require(_value>0)` check, users have to pass in at least 1 wei to extend the lock. This is problematic for protocols such as convex finance, which deal with voting power and need to constantly keep their lock maxed out. This isnt possible in the current contract since users have to pay in a non-zero amount to extend their lock.

## Recommendation

Allow users to refresh their locks back to 5 years for free.

# L02 VotingEscrow has unimplemented delegation mechanism

## Description

The VotingEscrow contract has a `delegatee` role implying that users can delegate their votes to others on their behalf. This is further highlighted in the `increaseAmount` function where there's different logic based on whether the delegatee is the `msg.sender` or not.

```solidity
if (delegatee == msg.sender) {
    // Logic 1
} else {
    // Logic 2
}
```

However, during lock creation it can be seen that the delegtion mechanism is ultimately unused, and locks are created with the `msg.sender` as the delgatee always.

```solidity
locked_.delegatee = msg.sender;
```

There is no other delegation function either. This means that the delegation mechanism is unimplemented and users cannot delegate their votes to others.

## Recommendation

Since the contract does not intend to support delegation, the contract logic related to delgations should be removed to avoid confusion and lower complexity and gas costs.
