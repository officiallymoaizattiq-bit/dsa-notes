# 128. Longest Consecutive Sequence

## Problem

Given an unsorted array of integers `nums`, return the length of the longest consecutive elements sequence.

You must write an algorithm that runs in **O(n)** time.

### Example

```python
nums = [100,4,200,1,3,2]

Output: 4
```

Explanation:

```text
1 → 2 → 3 → 4
```

The longest consecutive sequence has length **4**.

---

# My Learning Journey

This problem was a major lesson in moving from a working solution to an optimal solution.

I first solved it using sorting, then gradually discovered why that approach was not optimal and how the O(n) set-based solution works.

---

## Initial Understanding

My first instinct was:

```text
Sort the numbers
↓
Walk through neighbors
↓
Count consecutive streaks
```

This works because consecutive numbers become adjacent after sorting.

---

# Key Takeaways

- Consecutive links are different from consecutive lengths.
- Final streaks often require special handling.
- Sets remove duplicates but destroy ordering.
- Sorting + scanning gives an O(n log n) solution.
- A number is a sequence start if `num - 1` is not in the set.
- The optimal solution does not sort.
- Only sequence starts should trigger exploration.
- Nested loops are not automatically O(n²).
- Amortized analysis matters.
- This is a classic "set + sequence start detection" interview pattern.

---

# NeetCode Solution

```python
class Solution:
    def longestConsecutive(self, nums: List[int]) -> int:
        numSet = set(nums)
        longest = 0

        for n in nums:
            if (n - 1) not in numSet:
                length = 0
                while (n + length) in numSet:
                    length += 1
                longest = max(length, longest)

        return longest
```

---

# Final Mental Model

```text
1 → 2 → 3 → 4

Find the start:
1

Then walk:

1
↓
2
↓
3
↓
4

Count length.
Update answer.
Skip everything that isn't a start.
```

This shift from **sorting numbers** to **finding sequence starts** is the core insight of the problem.
