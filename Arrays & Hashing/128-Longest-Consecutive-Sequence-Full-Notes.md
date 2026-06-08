# 128. Longest Consecutive Sequence

## Problem

Given an unsorted array of integers `nums`, return the length of the longest consecutive elements sequence.

You must write an algorithm that runs in **O(n)** time.

### Example

```python
nums = [100,4,200,1,3,2]
```

Output:

```python
4
```

Explanation:

```text
1 → 2 → 3 → 4
```

The longest consecutive sequence has length 4.

---

# My Learning Journey

This problem turned into a lesson about the difference between:

1. Building a correct solution.
2. Building an optimal solution.
3. Understanding why a nested-loop solution can still be O(n).

I started with a sorting-based approach because consecutive numbers become neighbors after sorting. That idea was correct, but several implementation bugs appeared before reaching a working solution.

---

# Initial Intuition

My first thought was:

```text
Sort
↓
Walk through neighbors
↓
Count streak lengths
↓
Return the longest streak
```

Example:

```text
Original:
100,4,200,1,3,2

Sorted:
1,2,3,4,100,200
```

Once sorted, consecutive numbers become adjacent and are easy to detect.

---

# Attempt 1

```python
class Solution:
    def longestConsecutive(self, nums: List[int]) -> int:
        nums=sorted(nums)
        prevnum=nums[0]
        count=0
        cons=set()
        for i in range(len(nums)):
            if nums[i] == prevnum+1:
                count += 1
                prevnum=nums[i]
            else :
                cons.add(count)
                count=0
        max=0
        for n in cons:
            if n > max:
                max=n
        return max
```

## What I Was Thinking

I wanted to:

1. Sort the array.
2. Count consecutive numbers.
3. Store streak lengths.
4. Return the largest streak.

---

## Bugs Found

### Bug 1 — Empty Array

```python
prevnum = nums[0]
```

Fails when:

```python
nums = []
```

because index 0 does not exist.

---

### Bug 2 — Counting Links Instead of Elements

For:

```text
1 → 2 → 3 → 4
```

I was counting:

```text
1→2
2→3
3→4
```

which equals:

```text
3 links
```

but the answer should be:

```text
4 elements
```

Visualization:

```text
1 --- 2 --- 3 --- 4

4 numbers
3 links
```

---

### Bug 3 — Final Streak Lost

For:

```python
[1,2,3,4]
```

the streak never breaks.

Because the streak only got saved when a break occurred, the final streak was never processed.

---

## What I Learned

The length of a consecutive sequence is:

```text
Number of elements
```

not:

```text
Number of successful jumps
```

---

# Attempt 2

I switched to tracking streak length directly and removing duplicates.

```python
nums = list(set(sorted(nums)))
```

---

## Important Discovery

I assumed:

```text
Sort
↓
Remove duplicates
↓
Still sorted
```

but that assumption was wrong.

---

## Why It Failed

Consider:

```python
sorted(nums)
```

Result:

```text
[1,2,3,4,100,200]
```

Then:

```python
set(...)
```

Result:

```text
{1,2,3,4,100,200}
```

A set does not preserve ordering.

After:

```python
list(set(...))
```

I might get:

```text
[1,100,2,200,3,4]
```

or another arbitrary order.

---

## Major Lesson

Sets are good for:

```text
Duplicate removal
Membership checks
```

Sets are bad for:

```text
Preserving order
Neighbor comparisons
```

---

# Accepted Sorting Solution

```python
class Solution:
    def longestConsecutive(self, nums: List[int]) -> int:
        if not nums:
            return 0

        nums = sorted(list(set(nums)))

        count = 1
        streak = 0

        for i in range(1, len(nums)):
            if nums[i] == nums[i - 1] + 1:
                count += 1
            else:
                streak = max(streak, count)
                count = 1

        streak = max(streak, count)

        return streak
```

---

# Why This Solution Works

## Traversal Logic

After sorting:

```text
1 2 3 4 100 200
```

we move from left to right comparing neighbors.

---

## Business Logic

Check:

```python
nums[i] == nums[i - 1] + 1
```

If true:

```text
Current streak grows
```

Otherwise:

```text
Streak ended
Update answer
Start new streak
```

---

# Complexity Analysis

## Sorting Solution

### Time Complexity

```text
O(n log n)
```

Sorting dominates.

### Space Complexity

```text
O(n)
```

because of the set and sorted copy.

---

# Moving Toward O(n)

The follow-up requires:

```text
O(n)
```

Sorting prevents that.

So the key question became:

> How can we find consecutive sequences without sorting?

---

# Breakthrough Insight: Sequence Starts

A number is a sequence start if:

```python
num - 1 not in numSet
```

Example:

```text
Set:
{100,4,200,1,3,2}
```

Check each number:

| Number | Previous Exists? | Start? |
|----------|----------|----------|
| 1 | 0 ❌ | ✅ |
| 2 | 1 ✅ | ❌ |
| 3 | 2 ✅ | ❌ |
| 4 | 3 ✅ | ❌ |
| 100 | 99 ❌ | ✅ |
| 200 | 199 ❌ | ✅ |

Sequence starts:

```text
1
100
200
```

---

# Visual Understanding

Instead of sorting:

```text
100,4,200,1,3,2
```

we think:

```text
1 → 2 → 3 → 4
```

The only true start is:

```text
1
```

because:

```text
0
```

does not exist.

---

# Understanding the Inner Loop

Start:

```text
1
```

Initialize:

```text
current = 1
streak = 1
```

Walk forward:

```text
2 exists?
YES
```

```text
current = 2
streak = 2
```

---

```text
3 exists?
YES
```

```text
current = 3
streak = 3
```

---

```text
4 exists?
YES
```

```text
current = 4
streak = 4
```

---

```text
5 exists?
NO
```

Stop.

Sequence length:

```text
4
```

---

# Why This Is Not O(n²)

Initially the nested loop looked suspicious.

```text
for ...
    while ...
```

appears to be:

```text
O(n²)
```

But that is not what happens.

---

## Key Observation

For:

```text
1 → 2 → 3 → 4
```

Only:

```text
1
```

starts exploration.

---

Numbers:

```text
2
3
4
```

are skipped because they already have predecessors.

Visualization:

```text
1 → 2 → 3 → 4

↑ explore

2
↑ skip

3
↑ skip

4
↑ skip
```

---

## Amortized Analysis

Every number participates in a sequence walk at most once.

Total work:

```text
Visit each number
+
Walk through each sequence element once
```

Therefore:

```text
O(n)
```

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

# Deep Understanding of the NeetCode Solution

The elegant part of this solution is:

```python
while (n + length) in numSet:
```

Instead of maintaining a moving pointer, it measures distance from the start.

Visualization:

```text
n = 1

Check:
1 + 0
1 + 1
1 + 2
1 + 3
```

which becomes:

```text
1
2
3
4
```

The variable:

```text
length
```

simultaneously acts as:

1. The streak length.
2. The offset from the start.

---

# Pattern Recognition

This problem teaches a classic interview pattern:

```text
Set
↓
Find Starts
↓
Expand Only From Starts
↓
Track Best Answer
```

Whenever you see:

```text
Consecutive numbers
Longest chain
Sequence detection
```

this pattern should come to mind.

---

# Common Mistakes

### Counting Links Instead of Elements

```text
1→2→3→4
```

Length is:

```text
4
```

not:

```text
3
```

---

### Forgetting Final Streak Update

Very common in sorting solutions.

---

### Assuming Sets Preserve Order

They do not.

---

### Exploring Every Number

Only sequence starts should trigger exploration.

---

# Interview Explanation (30 Seconds)

Convert the numbers into a set for O(1) lookups.

A number is the start of a sequence if its predecessor does not exist.

For every start, walk forward while consecutive numbers exist and compute the sequence length.

Track the maximum length.

Each number is explored at most once, so the total runtime is O(n).

---

# Key Takeaways

- Consecutive links are not consecutive lengths.
- Sets destroy ordering.
- Sorting gives an O(n log n) solution.
- Sequence starts are identified using `num - 1 not in set`.
- Only starts should trigger exploration.
- Nested loops are not automatically O(n²).
- Amortized analysis matters.
- This is a classic set-based sequence problem.

---

# Final Mental Model

```text
1 → 2 → 3 → 4

Find the start:
1

Walk forward:

1
↓
2
↓
3
↓
4

Count length.
Update answer.
Ignore numbers that are not starts.
```

The biggest insight from this problem was shifting from:

```text
Sort the numbers
```

to:

```text
Find sequence starts
```
