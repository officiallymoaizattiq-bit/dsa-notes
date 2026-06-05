# 1. Two Sum

## Problem

Given an array of integers `nums` and an integer `target`, return indices of the two numbers such that they add up to `target`. 

You may assume that each input would have exactly one solution, and you may not use the same element twice. Return the answer with the smaller index first.

---

# My Learning Journey

This was my introduction to optimizing with Hash Maps.

While solving it, I learned:

* Nested loops often lead to Time Limit Exceeded (TLE) on LeetCode.
* Starting an inner loop at `i + 1` is crucial to avoid checking the same element twice.
* Hash maps (dictionaries) can reduce $O(n^2)$ time complexity to $O(n)$.
* In Python, `{i}` creates a Set, not an integer.
* Building a hash map *while* iterating (One-Pass) naturally prevents reusing the same element compared to building it beforehand (Two-Pass).
* The testing platform requires indices to be returned in a specific order (ascending).

---

# Solution 1: Brute Force (My First Attempt)

## Idea

Use a nested loop to check every possible combination of two numbers in the array.

## Code

```python
class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        for i in range(len(nums)):
            for j in range(1,len(nums)):
                if nums[i]+nums[j] == target:
                    return [i,j]
```

## Result

Failed due to logical bug.

## Why It Failed

The inner loop always started at `j = 1`. If the target was `4` and `nums = [3, 2, 4]`, the code would look at the `2` (index 1), add it to itself, and return `[1, 1]`. The problem states we cannot use the same element twice.

## Complexity

Time: O(n²)

Space: O(1)

## Lessons Learned

* Be extremely careful with loop ranges when comparing elements in the same array.

---

# Solution 2: Corrected Brute Force

## Idea

Fix the inner loop's starting range to start at `i + 1` to guarantee unique pairs.

## Code

```python
class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        for i in range(len(nums)):
            for j in range(i+1,len(nums)):
                if nums[i]+nums[j] == target:
                    return [i,j]
```

## Result

Failed with:

```text
Time Limit Exceeded
```

## Complexity

Time: O(n²)

Space: O(1)

## Why It Failed

The logic is 100% correct, but an $O(n^2)$ algorithm is too slow for arrays with tens of thousands of elements.

---

# Solution 3: Two-Pass Hash Map

## Idea

Use a Hash Map to store the numbers and their indices.
First loop: build the dictionary.
Second loop: calculate the `NumNeeded` (`target - current_number`) and check if it exists in the dictionary.

## Code

```python
class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        NumberMap={}

        for i in range(len(nums)):
            NumberMap[nums[i]]=i
            
        for i in range(len(nums)):
            NumNeeded=target-nums[i]
            if NumNeeded in NumberMap and i != NumberMap[NumNeeded]:
                return [i,NumberMap[NumNeeded]]
```

## Complexity

Time: O(n)

Space: O(n)

## Issues Encountered & Debugged

1. **Syntax Error:** I initially wrote `NumberMap[nums[i]] = {i}`. This created a Python Set, causing a `TypeError`. I fixed it to `= i`.
2. **Logic Error:** Because the whole dictionary was built first, `if NumNeeded in NumberMap` would sometimes find the current number itself. I had to explicitly add `and i != NumberMap[NumNeeded]`.

---

# Solution 4: One-Pass Hash Map (My Optimal Solution)

## Idea

Build the dictionary *as we go*. 
For each number, check if `NumNeeded` is in the map.
If yes, return the pair.
If no, add the current number to the map.

Checking before adding makes it mathematically impossible to match a number with itself.

## Code

```python
class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        NumberMap={}

        for i in range(len(nums)):
            NumNeeded=target-nums[i]
            
            if NumNeeded in NumberMap:
                return [NumberMap[NumNeeded], i]
                
            NumberMap[nums[i]]=i
```

## Complexity

Time: O(n)

Space: O(n)

## Why This Is Optimal

* Only requires a single loop, halving the execution time.
* Eliminates the clunky `i != NumberMap[NumNeeded]` check.
* **Note:** I had to swap the return order to `[NumberMap[NumNeeded], i]` because LeetCode expected the smaller index first.

---

# NeetCode Hash Map Solution

## Code

```python
class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        prevMap = {} # val : index

        for i, n in enumerate(nums):
            diff = target - n
            if diff in prevMap:
                return [prevMap[diff], i]
            prevMap[n] = i
```

## Complexity

Time: O(n)

Space: O(n)

## Difference from My Solution

NeetCode uses Python's built-in `enumerate(nums)` function.
This unpacks both the index (`i`) and the value (`n`) simultaneously. 
It behaves exactly like my One-Pass solution but reads slightly cleaner since you don't have to keep writing `nums[i]`.

---

# Complexity Comparison

| Solution                    | Time       | Space |
| --------------------------- | ---------- | ----- |
| Brute Force (Failed)        | O(n²)      | O(1)  |
| Corrected Brute Force       | O(n²)      | O(1)  |
| Two-Pass Hash Map           | O(n)       | O(n)  |
| One-Pass Hash Map           | O(n)       | O(n)  |
| NeetCode Solution           | O(n)       | O(n)  |

---

# Key Takeaways

1. Inner loops must be constrained properly (`i + 1`) to avoid duplicate elements.
2. Brute force $O(n^2)$ is rarely the accepted answer in interviews.
3. Thinking in terms of `target - current_number` unlocks the Hash Map strategy.
4. Hash maps provide $O(1)$ lookups, optimizing searches dramatically.
5. Building a map iteratively (One-Pass) is cleaner and safer than building it fully upfront (Two-Pass).
6. Python syntax is precise: `{i}` creates a Set, `i` is an integer.
7. Output format matters (LeetCode requires ascending index order).
8. `enumerate()` is highly useful for cleanly tracking indices and values simultaneously.