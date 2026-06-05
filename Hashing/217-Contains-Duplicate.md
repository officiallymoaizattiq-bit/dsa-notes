# 217. Contains Duplicate

## Problem

Given an integer array nums, return true if any value appears at least twice in the array.

---

## My First Attempt

### Idea

Heapify the array and compare adjacent elements.

### Code

```python
import heapq

class Solution:
    def hasDuplicate(self, nums):
        heapq.heapify(nums)

        for i in range(len(nums) - 1):
            if nums[i] == nums[i + 1]:
                return True

        return False
```

### Problems

1. Originally used `range(nums)` instead of `range(len(nums))`
2. Used `true` instead of `True`
3. `heapify()` does not sort the array
4. Adjacent duplicates are not guaranteed

### Complexity

Time: O(n)

Space: O(1)

### Why It Fails

Heap property != sorted order.

---

## Efficient Solution #1

### Using a Set

```python
class Solution:
    def hasDuplicate(self, nums):
        seen = set()

        for n in nums:
            if n in seen:
                return True

            seen.add(n)

        return False
```

### Walkthrough

nums = [1,2,3,1]

seen = {}

1 not in seen -> add
2 not in seen -> add
3 not in seen -> add
1 already in seen -> return True

### Complexity

Time: O(n)

Space: O(n)

---

## Efficient Solution #2

### Compare Lengths

```python
class Solution:
    def hasDuplicate(self, nums):
        return len(nums) != len(set(nums))
```

### Why It Works

Sets remove duplicates automatically.

Example:

nums = [1,2,3,1]

len(nums) = 4

len(set(nums)) = 3

4 != 3 -> duplicate exists

### Complexity

Time: O(n)

Space: O(n)

---

## Lessons Learned

- Sets provide O(1) average lookup.
- Heapify does not sort.
- A solution can be simple and still optimal.
