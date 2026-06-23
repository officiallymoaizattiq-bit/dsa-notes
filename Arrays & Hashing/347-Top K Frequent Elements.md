# 347. Top K Frequent Elements

## Problem

Given an integer array `nums` and an integer `k`, return the `k` most frequent elements. The answer can be returned in any order.

**Example:** `nums = [1,1,1,2,2,3]`, `k = 2` → `[1, 2]` (1 appears 3x, 2 appears 2x, those are the top two).

---

# My Learning Journey

## First Solve

Recognized the hash map immediately for frequency counting. Struggled early with `defaultdict` syntax and with the fact that I needed *multiple* top elements, not just the single max. Wrote a brute-force version that scanned the frequency map `k` times, grabbing the current max and zeroing it out each pass — accepted, but the nested scan made it slow on large `k`.

Optimized to a heap. Learned that Python's `heapq` is min-heap only, so I had to learn the negation trick — push `(-freq, num)` so the largest frequency floats to the top of a min-heap. Then reviewed NeetCode's bucket sort, which drops the log factor entirely by using array indices as frequency buckets for O(N).

## 25-Day Cold Revisit

Came back to this after 25 days and the recall was rough at first — which is exactly the point of spaced repetition. Rebuilt **both** the heap and the bucket sort solutions from scratch, no notes, debugging live.

What I recalled instantly: hash map for counting. What I had to rebuild: the heap tuple ordering, the negation mechanism, and the bucket-sort backwards walk. Hit several of the same bugs as the first solve (documented below) — most notably the heap-indexing bug, which I walked into a **second time**, confirming it as a genuine sticky spot.

---

# Attempt 1 (Original) — Manual Max Extraction (Brute Force)

## Idea

Count frequencies in a hash map. Loop `k` times. Each pass, scan the whole map for the current highest frequency, append that key, and reset its frequency to 0 so it won't be picked again.

## Code

```python
from collections import defaultdict

class Solution:
    def topKFrequent(self, nums: List[int], k: int) -> List[int]:
        outputList = []
        freqTable = defaultdict(int)

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

**Result:** Accepted (but suboptimal)

**Issues hit:** Tried initializing the max tracker with `-inf` (NameError — it's `float('-inf')` in Python). Tried `max(freqTable.values())` which returns the frequency, not the key. Works, but scanning all unique elements `k` times is the bottleneck.

---

# Attempt 2 (Original + Revisit) — Max-Heap

## Idea

Count frequencies, push every `(-freq, num)` tuple into a min-heap, pop `k` times. The negation turns the min-heap into a fake max-heap.

## Final Code

```python
from collections import defaultdict
import heapq

class Solution:
    def topKFrequent(self, nums: List[int], k: int) -> List[int]:
        hashmap = defaultdict(int)
        for n in nums:
            hashmap[n] += 1

        maxHeap = []
        for n, f in hashmap.items():
            heapq.heappush(maxHeap, (-f, n))

        output = []
        for i in range(k):
            output.append(heapq.heappop(maxHeap)[1])

        return output
```

**Result:** Accepted

## Bugs Hit During the 25-Day Revisit

| Bug | What I Wrote | Fix | Type |
|-----|-------------|-----|------|
| Used `range` to iterate the dict | `for n in range hashMap` | `for n, f in hashmap.items()` | Syntax / Implementation |
| Pushed only the frequency, lost the number | `heappush(maxHeap, hashMap[n])` | `heappush(maxHeap, (-f, n))` | Implementation |
| Variable name mismatch | `hashmap` vs `hashMap` | pick one consistently | Syntax |
| **Indexed into the heap** | `output.append(maxHeap[i])` | `output.append(heapq.heappop(maxHeap)[1])` | Logic |
| Wrong tuple index | `heappop(maxHeap)[0]` (returned negated freq) | `heappop(maxHeap)[1]` (returns the num) | Logic |

> ⚠️ **The `maxHeap[i]` indexing bug is a repeat.** My original notes flagged it in "Issues Encountered." I hit the exact same wall 25 days later. A heap is only *partially* ordered — `maxHeap[0]` is the smallest, but `maxHeap[1]`, `maxHeap[2]`... are **not** sorted. You must `heappop` to extract in order, because popping is what rebalances the heap.

---

# Attempt 3 (Original + Revisit) — Bucket Sort

## Idea

Instead of sorting frequencies, use the frequency as an **array index**. A number can't appear more than `len(nums)` times, so build an array of that length where each slot holds the list of numbers with that frequency. Walk it backwards to collect the top `k`.

## Final Code

```python
class Solution:
    def topKFrequent(self, nums: List[int], k: int) -> List[int]:
        freq = [[] for i in range(len(nums) + 1)]
        count = {}

        for n in nums:
            count[n] = 1 + count.get(n, 0)
        for n, f in count.items():
            freq[f].append(n)

        final = []
        for i in range(len(freq) - 1, 0, -1):
            for n in freq[i]:
                final.append(n)
                if len(final) == k:
                    return final
```

**Result:** Accepted — 21/21 test cases, 47ms, 8MB.

## Bugs Hit During the 25-Day Revisit

| Bug | What I Wrote | Fix | Type |
|-----|-------------|-----|------|
| Looped the dict directly, only got keys | `for n, f in count:` | `for n, f in count.items()` | Implementation |
| Off-by-one on the backwards walk **start** | `range(len(freq), 0, -1)` | `range(len(freq) - 1, 0, -1)` | Edge Case |

> ⚠️ **The `range` start off-by-one.** I knew the `-1` belonged somewhere — I put it on the *step* slot (which was already `-1`) instead of the *start*. `range(start, stop, step)`: start at `len(freq) - 1` (the last valid index, since arrays are 0-indexed), stop at `0` exclusive (skips the empty index-0 bucket), step `-1`.

## Setup Questions I Reasoned Through Correctly (Revisit)

- **Array length?** `len(nums) + 1` — the `+1` so index `len(nums)` exists for the case where one number fills the whole array.
- **What goes in each slot?** A **list**, not a tuple — tuples are immutable and you need to `append` since multiple numbers can share a frequency.
- **Stop condition?** When `len(final) == k`.

---

# Why It Works

**Heap:** A min-heap always pops the smallest element. By negating frequencies, the largest frequency becomes the most negative number, so it's treated as "smallest" and pops first. Popping `k` times yields the `k` most frequent. The number rides in slot `[1]` of the tuple so you never lose track of which number a frequency belongs to.

**Bucket Sort:** The key insight is that frequencies are bounded — no number can appear more than `len(nums)` times. That means you can use frequency as a direct array index instead of paying to sort. Drop each number into the bucket matching its count, then read buckets from highest index down. No comparisons, no log factor.

---

# Traversal Logic

**Heap:**
- Iterate `nums` once to build the count map.
- Iterate the count map's `.items()` to push tuples.
- Pop the heap `k` times.

**Bucket Sort:**
- Iterate `nums` once to count.
- Iterate the count map's `.items()` to place numbers into buckets.
- Walk the bucket array **backwards** (high freq → low), collecting until `k` reached.

---

# Business Logic

**Heap:**
- Negate frequency before pushing (`-f`) so the min-heap behaves like a max-heap.
- On pop, extract index `[1]` (the number), discard `[0]` (the negated frequency).

**Bucket Sort:**
- Append number to `freq[its_count]`.
- Stop the moment `len(final) == k` and return immediately.

---

# Variable Roles

## Heap

| Variable | Represents | Stores | How It Changes | Why It Exists |
|----------|-----------|--------|----------------|---------------|
| `hashmap` | frequency table | `{num: count}` | `+= 1` per occurrence | counts how often each number appears |
| `maxHeap` | priority queue | list of `(-freq, num)` tuples | pushed once per unique num, popped `k` times | extracts top-k efficiently |
| `output` | answer list | list of numbers | appended `k` times | holds the result |

## Bucket Sort

| Variable | Represents | Stores | How It Changes | Why It Exists |
|----------|-----------|--------|----------------|---------------|
| `count` | frequency table | `{num: count}` | `1 + count.get(n, 0)` per occurrence | counts occurrences |
| `freq` | buckets indexed by frequency | list of lists; `freq[c]` = nums appearing `c` times | numbers appended into matching index | replaces sorting with index placement |
| `final` | answer list | list of numbers | appended on backwards walk | holds the result, triggers early return |

---

# Visual Walkthrough — Bucket Sort

Input: `nums = [1,1,1,2,2,3]`, `k = 2`

**After counting:** `count = {1: 3, 2: 2, 3: 1}`

**Building the bucket array** (`freq`, length `len(nums) + 1 = 7`):

```
index:   0    1     2     3     4    5    6
         []   [3]   [2]   [1]   []   []   []
                ↑     ↑     ↑
              3 once 2 twice 1 thrice
```

**Backwards walk** from index `6` down to index `1`:

| Step | i (index) | Bucket `freq[i]` | Action | `final` | len == k? |
|------|-----------|------------------|--------|---------|-----------|
| 1 | 6 | `[]` | nothing | `[]` | no |
| 2 | 5 | `[]` | nothing | `[]` | no |
| 3 | 4 | `[]` | nothing | `[]` | no |
| 4 | 3 | `[1]` | append 1 | `[1]` | no |
| 5 | 2 | `[2]` | append 2 | `[1, 2]` | **yes → return** |

Result: `[1, 2]`

---

# Optimization Journey

1. **Brute Force:** count freqs, scan the map `k` times grabbing the max each pass. O(N + k·U).
2. **Bottleneck:** re-scanning all unique elements on every one of the `k` passes.
3. **Observation:** a heap gives you the max without re-scanning — `O(log)` per extraction.
4. **Optimization (heap):** push all uniques as `(-freq, num)`, pop `k` times. O(N + U log U).
5. **Further observation:** frequencies are bounded by `len(nums)`, so they can index an array directly — no sorting needed at all.
6. **Final approach (bucket sort):** index = frequency, walk backwards. O(N).

---

# Pattern Recognition

## Signal Phrases

- "k most frequent" / "top k"
- "frequency" / "count of elements"

## Pattern Used

Hash Map (counting) + Heap **or** Bucket Sort (extraction)

## Why This Pattern Fits

- "Top K" is the textbook signal for a **heap**.
- "Frequency" with a bounded range is the signal that unlocks **bucket sort** for linear time.

---

# Pattern Comparison

| Pattern | Why It Fits | Why It Doesn't | Verdict |
|---------|-------------|----------------|---------|
| Hash Map | needed to count freqs no matter what | doesn't extract top-k alone | required first step |
| Heap | "top k" is the classic heap signal | pays a log factor | strong, standard interview answer |
| Bucket Sort | freqs bounded by `len(nums)` → index trick | uses O(N) extra space | optimal time |
| Sorting | could sort by freq | O(U log U), beaten by both above | suboptimal |

---

# Common Mistakes

- Looping a dict directly (`for n, f in count`) — gives keys only, must use `.items()`. **(Hit twice in revisit, both solutions.)**
- Indexing into a heap (`maxHeap[i]`) instead of `heappop` — heaps are only partially ordered. **(Hit in original AND revisit.)**
- Grabbing the wrong tuple index — `[0]` is the negated freq, `[1]` is the number.
- Off-by-one on the bucket backwards walk — start at `len(freq) - 1`, not `len(freq)`.
- Pushing only the frequency into the heap and losing the number.

---

# Edge Cases

| Case | Input | Expected | How Handled |
|------|-------|----------|-------------|
| All same number | `[5,5,5], k=1` | `[5]` | needs index `len(nums)`, hence `+1` sizing |
| k == number of uniques | `[1,2,3], k=3` | `[1,2,3]` | walk collects all, early return on last |
| Single element | `[1], k=1` | `[1]` | one bucket, one pop |

---

# Debugging Lessons (From This Revisit)

- **My sticky spots are now named.** The `maxHeap[i]` heap-indexing bug is a confirmed repeat (original + revisit). The `range` start off-by-one in bucket sort is the new one. Naming them is how they stop recurring.
- **`.items()` is the reflex I keep dropping** when iterating dicts under recall pressure — missed it in both solutions this session.
- **Tuple index discipline:** sort-key goes in slot `[0]`, the thing you actually want returned goes in slot `[1]`.

---

# NeetCode / Official Solution — Bucket Sort

(Provided in my original notes file, included here for comparison.)

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

## Comparison to My Solution

Effectively identical — my revisit bucket sort converged on the same structure. The only difference is naming (`final`/`f` vs `res`/`c`). Good sign that the pattern is internalizing, not memorized line-by-line.

---

# Deep Understanding

The line that looks weird but is load-bearing: `range(len(freq) - 1, 0, -1)`.

- `len(freq) - 1` — the last **valid** index. `freq` has length `len(nums) + 1`, so its top index is `len(nums)`, i.e. `len(freq) - 1`. Starting at `len(freq)` is out of bounds.
- `0` (stop, exclusive) — index 0 is the "appeared zero times" bucket, always empty. Skip it.
- `-1` (step) — walk high frequency to low, so the most frequent come out first.

The whole O(N) win hinges on one fact: **a value can't appear more times than the array is long**, so frequency is a safe array index.

---

# Complexity Analysis

## Heap

| Metric | Complexity | Reason |
|--------|------------|--------|
| Time | O(N + U log U) | N to count; each of U uniques pushed/popped at log cost |
| Space | O(U) | heap holds one entry per unique element |

## Bucket Sort

| Metric | Complexity | Reason |
|--------|------------|--------|
| Time | **O(N)** | count is O(N), bucket placement is O(U) ≤ O(N), backwards walk visits ≤ N slots |
| Space | O(N) | the `freq` bucket array is sized `len(nums) + 1` |

**Why bucket sort beats the heap:** the heap pays an `O(log U)` factor on every push and pop to maintain order. Bucket sort throws that away entirely by exploiting the bounded frequency range — frequency becomes a direct array index, so the data is "sorted" by placement in strictly linear time. Same answer, no log factor.

---

# Related Problems

| # | Problem | Difficulty | Same Pattern? | Notes |
|---|---------|------------|---------------|-------|
| 692 | Top K Frequent Words | Medium | yes | heap with tie-breaking on lexical order |
| 215 | Kth Largest Element in an Array | Medium | yes (heap) | classic heap / quickselect |
| 451 | Sort Characters By Frequency | Medium | yes (bucket) | bucket sort by char frequency |
| 1 | Two Sum | Easy | hash map | same counting reflex |

---

# Interview Explanation (30 Seconds)

Count frequencies with a hash map, then extract the top k. The clean answer is a heap of `(-freq, num)` for O(N + U log U). The optimal answer is bucket sort — since no element appears more than `len(nums)` times, use frequency as an array index and walk it backwards for O(N).

---

# Interview Explanation (2 Minutes)

"Top k frequent" signals a frequency count plus a top-k extraction. First I build a hash map of counts in O(N). For extraction I have two options. A heap: push every unique element as `(-freq, num)` so Python's min-heap acts as a max-heap, then pop k times — that's O(N + U log U). But I can do better. Since a value can't appear more than `len(nums)` times, the frequency is bounded, so I make a bucket array where the index *is* the frequency, drop each number into its bucket, and walk from the highest index down collecting until I have k. That's O(N) time, O(N) space — the bounded-range observation is what removes the log factor. The tradeoff is the heap is more space-efficient at O(U) and generalizes to streaming, while bucket sort needs the full array up front.

---

# Common Follow-Up Questions

- **"What if the array is a stream / too big for memory?"** → heap wins; you can keep a bounded heap of size k, O(N log k), no need to materialize all buckets.
- **"Why `(-freq, num)` and not `(num, -freq)`?"** → the heap sorts by tuple element `[0]`; you want it sorting by frequency, so frequency must be first.
- **"What if there are ties at the k-th frequency?"** → problem allows any order, so either tied element is fine; if order mattered you'd add a tie-break key.

---

# Key Takeaways

- "Top K" → **heap** is the instinct; "frequency with bounded range" → **bucket sort** for O(N).
- Python `heapq` is min-heap only; negate the sort key to fake a max-heap.
- You **cannot index a heap** — `heappop` to extract in order. (My repeat bug.)
- Bucket sort backwards walk: `range(len(freq) - 1, 0, -1)`. (My off-by-one.)
- `.items()` when you need both key and value from a dict. (My recurring miss.)

---

# Revision Notes

**Re-reading in 1 week / 1 month — the minimum to refresh:**

1. Both solutions need a hash-map count first (identical start).
2. Heap: tuple is `(-freq, num)`, pop and take `[1]`. Don't index the heap.
3. Bucket: index = frequency, array length `len(nums) + 1`, walk `range(len(freq)-1, 0, -1)`, stop at `len(final) == k`.
4. Watch your two sticky spots: heap indexing, bucket range start.
5. Be able to say *why* bucket is O(N): bounded frequency range removes the log factor.

---

# Final Mental Model

Imagine sorting students by how many times they raised their hand. The heap is a teacher who keeps a running ranked list and reshuffles it every time a new tally comes in — correct, but constant reshuffling costs effort. Bucket sort is smarter: line up labeled boxes on a shelf, box "1" through box "however-many-students-there-are," and just drop each student into the box matching their hand-raise count. No ranking, no reshuffling — to find the most active, you start at the far end of the shelf and walk back. Because nobody can raise their hand more times than there are total moments, you always have exactly enough boxes. That bounded count is the whole trick.
