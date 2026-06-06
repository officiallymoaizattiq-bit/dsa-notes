# 347. Top K Frequent Elements

## Problem

Given an integer array `nums` and an integer `k`, return the `k` most frequent elements. You may return the answer in any order.

---

# My Learning Journey

My thinking evolved significantly during this problem. I immediately recognized the need for a hash map (dictionary) to count the frequencies. Initially, I struggled with `defaultdict` syntax and how to extract multiple elements instead of just the absolute maximum frequency. 

I wrote a brute-force approach that manually scanned the dictionary k times to find and extract the highest frequency, zeroing it out after each pass. While this was accepted by LeetCode, I realized the nested loops caused a time complexity bottleneck of roughly O(k * U) where U is unique elements. 

To optimize, I learned how to use Python's `heapq` library. Since Python only has Min-Heaps built-in, I learned the "Max-Heap Trick" of negating the frequencies `(-freq, num)` before pushing them into the heap, allowing me to pop the top k elements efficiently. Finally, I reviewed the NeetCode Bucket Sort approach, which achieves an O(N) time complexity by using array indices to represent frequencies.

---

# Solution 1: Manual Max Extraction (Brute Force)

## Idea
Count the frequencies using a hash map. Then, loop k times. On each iteration, scan the entire hash map to find the key with the current highest frequency. Append that key to the result list, and reset its frequency to 0 so it isn't picked again.

## Code
```python
from collections import defaultdict

class Solution:
    def topKFrequent(self, nums: List[int], k: int) -> List[int]:
        outputList=[]
        freqTable=defaultdict(int)

        for n in nums:
            freqTable[n] += 1
        
        for i in range(k):
            max_val = 0
            maxKey = 0
            for key, val in freqTable.items():
                if val > max_val:
                    max_val = val
                    maxKey = key
            outputList.append(maxKey)
            freqTable[maxKey] = 0
            
        return outputList

```

**Result**
Accepted

**Why It Failed / Issues Encountered**
Initially, I tried initializing the max tracker with `-inf`, which threw a `NameError` in Python. I also initially tried extracting the max using `max(freqTable.values())`, which only returned the frequency count, not the key. This solution works but is suboptimal because finding the max requires scanning all unique elements k times.

**Complexity**
Time: O(N + k * U) Space: O(U) *(Where U is the number of unique elements)*

**Lessons Learned**

* `defaultdict(int)` is the correct way to initialize a frequency map with 0s.
* Frequencies of elements in an array will always be >= 1, so tracking maximums can start at `0` instead of negative infinity.

---

# Solution 2: Max-Heap Optimization (My Final Solution)

## Idea

Count the frequencies, then push every element into a Priority Queue (Heap). Because Python's `heapq` is a min-heap, push tuples formatted as `(-frequency, number)` so the highest positive frequency becomes the smallest negative number, naturally floating to the top of the heap. Pop from the heap k times to get the answer.

## Code

```python
from collections import defaultdict
import heapq

class Solution:
    def topKFrequent(self, nums: List[int], k: int) -> List[int]:
        outputList=[]
        freqTable=defaultdict(int)
        maxHeap=[]

        for n in nums:
            freqTable[n] += 1
        
        for num, freq in freqTable.items():
            heapq.heappush(maxHeap, (-freq, num))

        for i in range(k):
            outputList.append(heapq.heappop(maxHeap)[1])

        return outputList

```

**Result**
Accepted

**Issues Encountered**
I initially tried to access the heap using indices `maxHeap[i]`, forgetting that a heap is only partially ordered and requires `heapq.heappop()` to correctly extract and rebalance. I also had to learn tuple unpacking `[1]` to extract just the number, not the negated frequency.

**Complexity**
Time: O(N + U log U) Space: O(U)

**Lessons Learned**

* Python's `heapq` only creates Min-Heaps.
* The "Max-Heap Trick" is multiplying the sort-key by `-1`.
* You can push tuples into a heap, and it will sort based on the first element of the tuple.

---

# Official / NeetCode Solution: Bucket Sort

## Code

```python
class Solution:
    def topKFrequent(self, nums: List[int], k: int) -> List[int]:
        count = {}
        freq = [[] for i in range(len(nums) + 1)]

        for n in nums:
            count[n] = 1 + count.get(n, 0)
        for n, c in count.items():
            freq[c].append(n)

        res = []
        for i in range(len(freq) - 1, 0, -1):
            for n in freq[i]:
                res.append(n)
                if len(res) == k:
                    return res

```

## Idea

Instead of sorting the frequencies, use an array where the **index** represents the frequency. Count the occurrences of each number, then append the number to the `freq` array at the index matching its count. Finally, iterate backward through the `freq` array (from highest frequency down to 1) and append elements to the result until `k` elements are collected.

## Complexity

Time: O(N) Space: O(N)

## Difference from My Solution

This completely avoids the O(log U) overhead of pushing and popping from a heap. By bounding the maximum possible frequency to the length of the input array `len(nums)`, we can use array indices to implicitly sort the data in strictly linear time.

---

## Complexity Comparison

| Solution | Time | Space | Notes |
| --- | --- | --- | --- |
| Brute Force | O(N + k * U) | O(U) | Nested loops cause slowdown on large k. |
| Max-Heap | O(N + U log U) | O(U) | Much faster, standard pattern for "Top K" problems. |
| Bucket Sort | O(N) | O(N) | Most optimal. Uses array indices as frequency buckets. |

## Key Takeaways

* `defaultdict(int)` or `dict.get(n, 0)` are essential for clean frequency counting.
* When asked for "Top K" or "Bottom K", a **Heap (Priority Queue)** should be the immediate instinct.
* To simulate a Max-Heap in Python, push tuples with negated values: `heapq.heappush(heap, (-value, item))`.
* If the range of values to sort is bounded (like frequency counts <= length of array), **Bucket Sort** is a powerful trick to achieve O(N) time.

## Tags

[Hash Map]
[Heap (Priority Queue)]
[Bucket Sort]

```

```