# QA Report

## **Table of Contents**

|       | Issue                                                                                                                 |
| ----- | --------------------------------------------------------------------------------------------------------------------- |
| QA-01 | `_remove_gauge_weight()` in GaugeController.sol should update points total  |
| QA-02 | Unnecessary write to history inside `_checkpoint()` of `VotingEscrow` when epoch is 0 |

## QA-01 `_remove_gauge_weight()` in GaugeController.sol should update points total

### Impact

`points_total` not immediately reflect the current value after the removal of gauge weight.

### Proof of Concept

It can be observed that inside ` _remove_gauge_weight()` function, it will only update `points_weight` of the gauge and `points_sum` of the gauge type, but not updating `points_total` :

```solidity
    function _remove_gauge_weight(address _gauge) internal {
        int128 gauge_type = gauge_types_[_gauge] - 1;
        uint256 next_time = ((block.timestamp + WEEK) / WEEK) * WEEK;

        uint256 old_weight_bias = _get_weight(_gauge);
        uint256 old_weight_slope = points_weight[_gauge][next_time].slope;
        uint256 old_sum_bias = _get_sum(gauge_type);

        points_weight[_gauge][next_time].bias = 0;
        points_weight[_gauge][next_time].slope = 0;

        uint256 new_sum = old_sum_bias - old_weight_bias;
        points_sum[gauge_type][next_time].bias = new_sum;
        points_sum[gauge_type][next_time].slope -= old_weight_slope;
        // We have to cancel all slope changes (gauge specific and global) that were caused by this gauge
        // This is not very efficient, but does the job for a governance function that is called very rarely
        for (uint256 i; i < 263; ++i) {
            uint256 time_to_check = next_time + i * WEEK;
            uint256 gauge_weight_change = changes_weight[_gauge][time_to_check];
            if (gauge_weight_change > 0) {
                changes_weight[_gauge][time_to_check] = 0;
                changes_sum[gauge_type][time_to_check] -= gauge_weight_change;
            }
        }
    }
```
After the operation, if `gauge_relative_weight` is queried by users without updating the total, it will return wrong value.

### Recommended Mitigation Steps

Update `points_total` inside `_remove_gauge_weight`

---

## QA-02 Unnecessary write to history inside `_checkpoint()` of `VotingEscrow` when epoch is 0

### Impact

Unnecessary saving `userOldPoint` to `userPointHistory[_addr][uEpoch + 1]` when `uEpoch` is 0.

### Proof of Concept

https://github.com/code-423n4/2024-03-neobase/blob/main/src/VotingEscrow.sol#L168-L171

Inside `_checkpoint()` function:

```solidity
    function _checkpoint(
        address _addr,
        LockedBalance memory _oldLocked,
        LockedBalance memory _newLocked
    ) internal {
        Point memory userOldPoint;
        Point memory userNewPoint;
        int128 oldSlopeDelta = 0;
        int128 newSlopeDelta = 0;
        uint256 epoch = globalEpoch;

        if (_addr != address(0)) {
            // Calculate slopes and biases
            // Kept at zero when they have to
            if (_oldLocked.end > block.timestamp && _oldLocked.delegated > 0) {
                userOldPoint.slope = _oldLocked.delegated / int128(int256(LOCKTIME));
                userOldPoint.bias = userOldPoint.slope * int128(int256(_oldLocked.end - block.timestamp));
            }
            if (_newLocked.end > block.timestamp && _newLocked.delegated > 0) {
                userNewPoint.slope = _newLocked.delegated / int128(int256(LOCKTIME));
                userNewPoint.bias = userNewPoint.slope * int128(int256(_newLocked.end - block.timestamp));
            }

            // Moved from bottom final if statement to resolve stack too deep err
            // start {
            // Now handle user history
            uint256 uEpoch = userPointEpoch[_addr];
            if (uEpoch == 0) {
                userPointHistory[_addr][uEpoch + 1] = userOldPoint;
            }
            userPointEpoch[_addr] = uEpoch + 1;
            userNewPoint.ts = block.timestamp;
            userNewPoint.blk = block.number;
            userPointHistory[_addr][uEpoch + 1] = userNewPoint;
            // ...
}
```

It can be observed that `userOldPoint` is stored to `userPointHistory[_addr][uEpoch + 1]`, but it will overwritten with `userNewPoint` afterward.

### Recommended Mitigation Steps

Consider to remove the `uEpoch == 0` conditional action.

---