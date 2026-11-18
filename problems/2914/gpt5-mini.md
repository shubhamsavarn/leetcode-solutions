# [Problem 2914: Minimum Number of Changes to Make Binary String Beautiful](https://leetcode.com/problems/minimum-number-of-changes-to-make-binary-string-beautiful/description/?envType=daily-question)

## Initial thoughts (stream-of-consciousness)

The string needs to be split into even-length blocks where each block is all `0`s or all `1`s. Since the total length is even, a natural way to think about this is to group the string into fixed pairs: `(s[0], s[1])`, `(s[2], s[3])`, etc.

For any beautiful partition, consider one of these pairs. If a block covers both characters of a pair, that block must be uniform (all `0`s or all `1`s), so the two characters in the pair must end up equal. That means:
- If `s[i] == s[i+1]`, the pair is already fine — 0 changes needed.
- If `s[i] != s[i+1]`, at least one of them must change — exactly 1 change is enough to make them match.

Because the pairs are disjoint, decisions for one pair don't affect another. So the answer should just be the count of mismatched pairs among `(0,1), (2,3), ...`.

## Refining the problem, round 2 thoughts

**Lower bound:** Each mismatched pair `(s[i], s[i+1])` contains two different characters. Any valid beautiful substring containing both must be all `0`s or all `1`s, so at least one character in that pair must be changed. Thus we need at least 1 change per mismatched pair.

**Upper bound:** If we change exactly one character in every mismatched pair to match the other, every pair becomes `"00"` or `"11"`. We can then partition the string into these length-2 substrings, each of which is even-length and uniform. So the number of mismatched pairs is also sufficient.

**Edge cases:**
- Length 2: works directly.
- Already beautiful (e.g., `"0000"`): 0 mismatched pairs → 0 changes.
- Fully alternating (e.g., `"0101"`): every pair mismatches → need 1 change per pair.

**Complexity:** We scan the string once in steps of 2, so O(n) time and O(1) extra space.

## Attempted solution(s)

```python
class Solution:
    def minChanges(self, s: str) -> int:
        changes = 0
        # Process the string in non-overlapping pairs: (0,1), (2,3), ...
        for i in range(0, len(s), 2):
            if s[i] != s[i + 1]:
                changes += 1
        return changes
```

- **Approach:** Greedy pairing. Iterate over the string two characters at a time. Whenever the two characters in a pair differ, we need exactly one change to make them equal; otherwise no change is needed. Summing these gives the minimum total changes.
- **Time Complexity:** O(n), where n is the length of `s`.
- **Space Complexity:** O(1) — only a counter is used.
- **Correctness:** Each mismatched pair requires at least one change, and changing one character per mismatched pair is sufficient to make every pair uniform, allowing a valid partition into length-2 substrings.