# [Problem 2567: Minimum Score by Changing Two Elements](https://leetcode.com/problems/minimum-score-by-changing-two-elements/description/?envType=daily-question)

## Initial thoughts (stream-of-consciousness)

We can change exactly two elements to any integer values. The score is `high + low`, where:
- `high` = max absolute difference = `max(nums) - min(nums)` (after sorting)
- `low` = min absolute difference between any pair

Since we can set the two changed elements to whatever we want, a natural idea is to make them equal to each other. If we do that, `low = 0` (because `|x - x| = 0`), and the score becomes just `high = max - min`.

So the problem reduces to: **what is the smallest possible range (`max - min`) we can achieve by keeping `n-2` elements fixed and optimally placing the other two?**

To minimize range, we should keep elements that are clustered together. In a sorted array, that means removing elements from the extremes. There are only three sensible ways to remove 2 elements from the ends:
1. Remove the 2 largest → keep `nums[0..n-3]`
2. Remove 1 largest and 1 smallest → keep `nums[1..n-2]`
3. Remove the 2 smallest → keep `nums[2..n-1]`

For each choice, we set the two changed elements to some value inside the kept range (e.g., equal to each other), giving `low = 0` and `high = kept_max - kept_min`.

The answer is the minimum of these three ranges.

## Refining the problem, round 2 thoughts

**Edge cases:**
- `n = 3`: We keep only 1 element, so range is 0. We change the other two to match it, making all three equal → score 0. All three formulas give `0`, which is correct.
- Duplicate values: Sorting handles them naturally; range can be 0 if all kept elements are equal.

**Why not keep non-contiguous elements?**
Removing an interior element never reduces the range of the kept set (the min and max are determined by the extremes). So we only need to consider trimming from the left and/or right.

**Why is `low = 0` always achievable?**
Set both changed elements to the same integer `v` where `kept_min <= v <= kept_max`. Then we have at least one pair with difference 0, so `low = 0`, and `high = kept_max - kept_min`. Score = `kept_max - kept_min`. Since `score >= high >= range`, this is optimal.

**Complexity:** Sorting dominates at `O(n log n)`, which is fine for `n <= 10^5`. Space is `O(1)` extra (aside from sort).

## Attempted solution

```python
class Solution:
    def minimizeSum(self, nums: List[int]) -> int:
        nums.sort()
        n = len(nums)
        
        # Three ways to remove 2 elements from the extremes of the sorted array:
        # 1. Remove the 2 largest → keep nums[0 .. n-3]
        # 2. Remove 1 largest and 1 smallest → keep nums[1 .. n-2]
        # 3. Remove the 2 smallest → keep nums[2 .. n-1]
        #
        # For each, set the two changed elements to the same value inside the kept range,
        # making low = 0 and score = kept_max - kept_min.
        return min(
            nums[n - 3] - nums[0],
            nums[n - 2] - nums[1],
            nums[n - 1] - nums[2],
        )
```

- **Approach:** Sort the array. The minimum score equals the minimum range achievable by keeping `n-2` elements. Since trimming interior elements doesn't reduce range, only three candidates matter: drop 2 from the right, drop 1 from each side, or drop 2 from the left. For any chosen kept window, set both changed elements to the same value inside that window, forcing `low = 0` and making the score exactly the window's range.
- **Time Complexity:** `O(n log n)` due to sorting.
- **Space Complexity:** `O(1)` extra space (sorting is in-place).