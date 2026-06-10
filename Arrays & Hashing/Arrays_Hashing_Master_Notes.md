# Master Notes — Arrays & Hashing (NeetCode 150)

> A single reference doc covering all 9 problems in the section. Read this before any Arrays & Hashing problem. If you can answer the self-test at the bottom, you own this section.

---

# Table of Contents

1. [Why This Section Matters](#why-this-section-matters)
2. [The Core Mental Model](#the-core-mental-model)
3. [Pattern Recognition Decision Tree](#pattern-recognition-decision-tree)
4. [Python Syntax Cheat Sheet](#python-syntax-cheat-sheet)
5. [The 7 Core Patterns](#the-7-core-patterns)
   - [Pattern 1: Set for Membership / Duplicate Detection](#pattern-1-set-for-membership--duplicate-detection)
   - [Pattern 2: Hash Map Frequency Counting](#pattern-2-hash-map-frequency-counting)
   - [Pattern 3: Complement Lookup](#pattern-3-complement-lookup)
   - [Pattern 4: Canonical Key Grouping](#pattern-4-canonical-key-grouping)
   - [Pattern 5: Prefix and Suffix Accumulation](#pattern-5-prefix-and-suffix-accumulation)
   - [Pattern 6: Bucket Sort by Bounded Value](#pattern-6-bucket-sort-by-bounded-value)
   - [Pattern 7: Sequence Start Detection](#pattern-7-sequence-start-detection)
6. [Auxiliary Pattern: Length-Prefix Encoding](#auxiliary-pattern-length-prefix-encoding)
7. [Complexity Summary](#complexity-summary)
8. [Problem-to-Pattern Map](#problem-to-pattern-map)
9. [Common Bugs Across the Section](#common-bugs-across-the-section)
10. [Interview Strategy](#interview-strategy)
11. [Self-Test](#self-test)

---

# Why This Section Matters

Arrays & Hashing is the foundation of ~40% of all interview problems. Almost every other section (Two Pointers, Sliding Window, Stack, Graphs) reuses these techniques as subroutines.

The single biggest idea in this section:

> **Hashing trades space for time. You add O(n) space to remove an O(n) factor from time.**

Brute force is almost always O(n²) because you scan the array repeatedly. Hashing gives you O(1) lookups, which collapses one of those loops.

---

# The Core Mental Model

When you see an array problem, ask in order:

```text
1. Do I need to know "have I seen this before"?     → SET
2. Do I need to know "how many of this exist"?      → HASH MAP (frequency count)
3. Do I need to know "where is this value"?         → HASH MAP (value → index)
4. Do I need to group similar things together?      → HASH MAP (canonical key → list)
5. Do I need running totals from both directions?   → PREFIX + SUFFIX arrays
6. Are the values bounded by something small?       → BUCKET / INDEX-AS-VALUE
7. Do I need consecutive/sequential structure?      → SET + START DETECTION
```

Most Arrays & Hashing problems are solved by exactly one of these seven ideas. A few combine two.

---

# Pattern Recognition Decision Tree

```text
Given an array problem:

├── "Are there duplicates?"
│   └── Pattern 1: Set
│
├── "Are these two collections equal in some way?"
│   └── Pattern 2: Frequency count + compare
│
├── "Find two elements that combine to a target"
│   └── Pattern 3: Complement lookup
│
├── "Group these together / are these equivalent under some transform"
│   └── Pattern 4: Canonical key grouping
│
├── "Compute something for every index using all other indices"
│   └── Pattern 5: Prefix + Suffix
│
├── "Top K / sort by frequency where freq ≤ n"
│   └── Pattern 6: Bucket sort
│
├── "Longest run / consecutive sequence"
│   └── Pattern 7: Set + start detection
│
└── "Serialize / parse variable-length strings"
    └── Aux Pattern: Length-prefix encoding
```

---

# Python Syntax Cheat Sheet

The 95% of syntax you need for this section.

## Set operations

```python
s = set()
s.add(x)              # add
s.remove(x)           # remove (raises KeyError if missing)
s.discard(x)          # remove (no error if missing)
x in s                # O(1) membership check
len(s)                # size
s1 & s2               # intersection
s1 | s2               # union
s1 - s2               # difference

# Build set from iterable
s = set([1, 2, 3])
s = set(nums)         # all unique values
```

## Dict / hash map operations

```python
d = {}
d[key] = value
d.get(key, default)   # safe lookup
key in d              # O(1) check
del d[key]
d.items()             # iterable of (key, value)
d.keys()
d.values()

# defaultdict — auto-initializes missing keys
from collections import defaultdict
d = defaultdict(int)   # missing keys → 0
d = defaultdict(list)  # missing keys → []
d = defaultdict(set)   # missing keys → set()

# Counter — auto frequency map
from collections import Counter
c = Counter("hello")   # {'l': 2, 'h': 1, 'e': 1, 'o': 1}
c.most_common(2)       # [('l', 2), ('h', 1)]
c == Counter("ohell")  # True (anagram check in one line)
```

## Heap (min-heap is default)

```python
import heapq
heap = []
heapq.heappush(heap, x)       # push
heapq.heappop(heap)           # pop smallest
heap[0]                       # peek smallest (no pop)
heapq.heapify(lst)            # convert list to heap in place, O(n)

# Max-heap trick: negate values
heapq.heappush(heap, -x)
-heapq.heappop(heap)          # gets back the max

# Heap of tuples — sorted by first element
heapq.heappush(heap, (-freq, num))
```

## Sorting

```python
sorted(nums)                  # returns new sorted list
nums.sort()                   # in-place
sorted(nums, reverse=True)
sorted(nums, key=lambda x: x[1])
sorted(strs, key=lambda s: (len(s), s))   # multi-key
```

## Tuple unpacking & enumerate

```python
for i, n in enumerate(nums):  # index + value
for k, v in d.items():        # key + value

a, b = (1, 2)                 # unpacking
a, b = b, a                   # swap, no temp variable
```

## Hashable trick — list to tuple

```python
# Lists are unhashable, can't be dict keys
key = tuple(sorted(s))        # sortable, hashable
key = tuple([0,1,2,3])        # tuple from list
```

## ASCII / character math

```python
ord('a')                      # 97
chr(97)                       # 'a'
ord(c) - ord('a')             # maps 'a'-'z' to 0-25
```

## List comprehensions

```python
res = [0] * n                 # init n zeros
res = [[] for _ in range(n)]  # init n empty lists (use this, NOT [[]] * n)
res = [x*x for x in nums]
res = [x for x in nums if x > 0]
```

> **CRITICAL:** `[[]] * n` creates n references to the SAME list. Use `[[] for _ in range(n)]` for independent lists.

---

# The 7 Core Patterns

---

## Pattern 1: Set for Membership / Duplicate Detection

### Signal phrases

- "Are there any duplicates?"
- "Has this value been seen before?"
- "Do all elements appear at most once?"
- "Does this value exist in some collection?"

### When to use

Whenever the question reduces to *"have I seen this before?"* — that's a set. Sets give O(1) average lookup.

### Template

```python
def has_duplicate(nums):
    seen = set()
    for n in nums:
        if n in seen:
            return True
        seen.add(n)
    return False
```

### Variations

**1. One-line existence check:**
```python
return len(nums) != len(set(nums))
```

**2. Multiple independent sets** (one per row/col/region):
```python
rows = [set() for _ in range(9)]
cols = [set() for _ in range(9)]
```

**3. Composite key sets** (when you need to track 2D locations):
```python
boxes = defaultdict(set)
boxes[(r // 3, c // 3)].add(value)
```

### Example problems

**#217 Contains Duplicate** — direct application. Walk array, check if seen, add if not.

**#36 Valid Sudoku** — three independent set families: 9 rows, 9 cols, 9 boxes. Same membership check, different traversal. The hard part is mapping `(r, c)` to its box via `(r // 3, c // 3)`.

**#128 Longest Consecutive Sequence** — set used for O(1) "does this number exist" checks during sequence walks (see Pattern 7).

### Common bugs

- Using a list instead of a set → `in` becomes O(n), total O(n²)
- Forgetting to add elements to the set
- Using `set([...])` repeatedly inside a loop (rebuilds the set every iteration — disaster)

---

## Pattern 2: Hash Map Frequency Counting

### Signal phrases

- "How many times does X appear?"
- "Are these two collections the same set of elements?"
- "Find the most/least common element"
- "Are two strings anagrams of each other?"

### When to use

Whenever counts of elements matter, not their positions.

### Template

```python
from collections import defaultdict

def count_frequencies(nums):
    count = defaultdict(int)
    for n in nums:
        count[n] += 1
    return count
```

### Variations

**1. defaultdict approach:**
```python
count = defaultdict(int)
for c in s:
    count[c] += 1
```

**2. Counter (idiomatic Python):**
```python
from collections import Counter
count = Counter(s)
return Counter(s) == Counter(t)   # one-line anagram check
```

**3. Fixed-size array** (when domain is bounded, like 26 letters):
```python
count = [0] * 26
for c in s:
    count[ord(c) - ord('a')] += 1
```

**4. Manual dict with `.get()`:**
```python
count = {}
for c in s:
    count[c] = 1 + count.get(c, 0)
```

### Example problems

**#242 Valid Anagram** — count chars in both strings, compare. Two valid approaches:
- `Counter(s) == Counter(t)` — one line
- Build two `defaultdict(int)` and compare — interview-friendly

**#347 Top K Frequent Elements** — step 1 is always frequency counting; what differs is how you extract the top K.

**#49 Group Anagrams** — frequency arrays of length 26 become canonical keys (see Pattern 4).

### Common bugs

- Forgetting that `defaultdict(int)` requires the import
- Using `count[key] = count[key] + 1` without checking existence first → KeyError
- Comparing dicts with `==` when one has zero-count entries the other doesn't (rare but possible)

---

## Pattern 3: Complement Lookup

### Signal phrases

- "Find two numbers that sum to a target"
- "Find pairs / triplets that satisfy some relation"
- "Does there exist X such that f(X, current) = target?"

### When to use

When you're searching for an element that pairs with the current one to satisfy a condition. Instead of nested loops (O(n²)), compute "what would I need?" and look it up in a hash map (O(n)).

### Template — One-Pass

```python
def two_sum(nums, target):
    seen = {}                          # value → index
    for i, n in enumerate(nums):
        complement = target - n
        if complement in seen:
            return [seen[complement], i]
        seen[n] = i
    return []
```

**Key insight:** Check FIRST, then add. This prevents matching an element with itself.

### Variations

**1. Two-pass (less elegant, more error-prone):**
```python
# Build map first
seen = {n: i for i, n in enumerate(nums)}
# Search after
for i, n in enumerate(nums):
    complement = target - n
    if complement in seen and seen[complement] != i:  # need this check!
        return [i, seen[complement]]
```

**2. Pair existence (no need for indices):**
```python
seen = set()
for n in nums:
    if target - n in seen:
        return True
    seen.add(n)
return False
```

### Example problems

**#1 Two Sum** — canonical example. The pattern: for each element, ask "what value would complete this?" and look it up.

This pattern generalizes to:
- 3Sum (sort + outer loop + 2Sum inside)
- 4Sum (sort + 2 outer loops + 2Sum)
- Subarray Sum Equals K (prefix sums + complement lookup)

### Common bugs

- **Order matters:** adding to map BEFORE checking lets an element match itself. Always check first.
- **Index order:** problems may require the smaller index first — handle the order of `[seen[complement], i]` accordingly.
- **Using a set instead of a dict** when you need the index — sets don't store positions.

---

## Pattern 4: Canonical Key Grouping

### Signal phrases

- "Group X together where they share property Y"
- "Find all sets of items equivalent under transformation T"
- "Cluster these items"

### When to use

When items are "equivalent" under some transformation, and you want to bucket equivalent items together. The transformation produces a **canonical key** that's identical for all members of a group.

### Template

```python
from collections import defaultdict

def group_by_canonical_key(items):
    groups = defaultdict(list)
    for item in items:
        key = canonicalize(item)        # the transformation
        groups[key].append(item)
    return list(groups.values())
```

### Two ways to canonicalize anagrams

**1. Sort the string** — O(n log n) per item, intuitive:
```python
key = tuple(sorted(s))     # MUST be tuple (hashable), not list
```

**2. Frequency vector** — O(n) per item, optimal:
```python
count = [0] * 26
for c in s:
    count[ord(c) - ord('a')] += 1
key = tuple(count)         # tuple of 26 ints
```

### Example problems

**#49 Group Anagrams** — anagrams share the same sorted string OR the same character frequency vector.

This pattern also shows up in:
- Group shifted strings (canonical = pattern of differences)
- Group strings by length (canonical = `len(s)`)
- Group numbers by digit sum (canonical = `sum_of_digits(n)`)

### Common bugs

- **Using a list as a key** → `TypeError: unhashable type: 'list'`. Convert to tuple.
- **Using `.append()` with assignment**: `d[k] = d[k].append(x)` returns None. Use `d[k].append(x)` (no assignment), and use `defaultdict(list)` so the key auto-initializes.
- **Returning a generator** when problem wants a list: wrap in `list(...)`.

---

## Pattern 5: Prefix and Suffix Accumulation

### Signal phrases

- "For each index, compute X using all other elements"
- "Without using division"
- "Cumulative sum / product / max from left and right"
- "Running totals"

### When to use

When the answer at index `i` depends on values both BEFORE and AFTER `i`. Brute force is O(n²) — for each index, scan the rest. Prefix/suffix arrays make this O(n).

### Template

```python
def product_except_self(nums):
    n = len(nums)
    res = [1] * n
    
    # Prefix pass: res[i] = product of everything LEFT of i
    prefix = 1
    for i in range(n):
        res[i] = prefix
        prefix *= nums[i]
    
    # Suffix pass: multiply in everything RIGHT of i
    suffix = 1
    for i in range(n - 1, -1, -1):
        res[i] *= suffix
        suffix *= nums[i]
    
    return res
```

### How it works (visual)

For `nums = [1, 2, 3, 4]`:

```
After prefix pass:
  res[0] = 1                    (nothing left of 0)
  res[1] = 1                    (left of 1: just nums[0]=1)
  res[2] = 1*2 = 2              (left of 2: nums[0]*nums[1])
  res[3] = 1*2*3 = 6            (left of 3: nums[0]*nums[1]*nums[2])

After suffix pass (multiplying in right products):
  res[3] *= 1 = 6               (nothing right of 3)
  res[2] *= 4 = 8               (right of 2: nums[3]=4)
  res[1] *= 4*3 = 12            (right of 1: nums[3]*nums[2])
  res[0] *= 4*3*2 = 24          (right of 0: nums[3]*nums[2]*nums[1])
```

### Variations

**1. Two separate arrays** (cleaner but uses more space):
```python
prefix = [1] * n
suffix = [1] * n
for i in range(1, n):
    prefix[i] = prefix[i-1] * nums[i-1]
for i in range(n-2, -1, -1):
    suffix[i] = suffix[i+1] * nums[i+1]
return [prefix[i] * suffix[i] for i in range(n)]
```

**2. Prefix sum** (same pattern with addition):
```python
prefix_sum = [0] * (n + 1)
for i in range(n):
    prefix_sum[i+1] = prefix_sum[i] + nums[i]
# Sum of nums[l..r] inclusive = prefix_sum[r+1] - prefix_sum[l]
```

### Example problems

**#238 Product of Array Except Self** — direct application.

This pattern generalizes to:
- Range sum queries (prefix sum)
- Maximum subarray (Kadane's = prefix max - prefix min)
- Trapping Rain Water (left max + right max at each position)
- Best Time to Buy/Sell Stock (running min from left)

### Common bugs

- **Off-by-one in reverse range:** `range(n-2, 0, -1)` skips index 0. Use `range(n-1, -1, -1)`.
- **Confusing "neighbor" with "cumulative":** `nums[i] * nums[i-1]` is just two adjacent values, not the product of everything to the left.
- **Modifying the result array in the prefix pass and forgetting to reset:** always be explicit about whether `res[i]` is "prefix-so-far" or "final answer."

---

## Pattern 6: Bucket Sort by Bounded Value

### Signal phrases

- "Top K most frequent"
- "Find the K most common"
- "Sort by frequency" when frequencies are bounded by `n`

### When to use

When the values you're sorting by are bounded by a small range (like frequencies in an array of length `n` can't exceed `n`). You can use array indices to represent values, achieving O(n) instead of O(n log n).

### Template

```python
def top_k_frequent(nums, k):
    count = Counter(nums)
    
    # bucket[i] = list of numbers with frequency i
    buckets = [[] for _ in range(len(nums) + 1)]
    for num, freq in count.items():
        buckets[freq].append(num)
    
    # Walk from highest frequency down
    res = []
    for freq in range(len(buckets) - 1, 0, -1):
        for num in buckets[freq]:
            res.append(num)
            if len(res) == k:
                return res
    return res
```

### Why this works

The maximum possible frequency in an array of length `n` is `n` itself (when all elements are equal). So we create `n + 1` buckets indexed by frequency.

For `nums = [1,1,1,2,2,3]`:
```
Frequencies: {1: 3, 2: 2, 3: 1}

buckets[0] = []
buckets[1] = [3]
buckets[2] = [2]
buckets[3] = [1]
buckets[4] = []
...

Walking right-to-left: 1, 2, 3 → top K=2 is [1, 2]
```

### Alternative: Heap (slightly slower but more general)

```python
import heapq

def top_k_frequent_heap(nums, k):
    count = Counter(nums)
    heap = []
    for num, freq in count.items():
        heapq.heappush(heap, (-freq, num))   # negate for max-heap
    return [heapq.heappop(heap)[1] for _ in range(k)]
```

Time: O(n + U log U) where U is unique elements. Bucket sort beats this with O(n).

### Example problems

**#347 Top K Frequent Elements** — direct application. Heap works too, bucket sort is optimal.

### Common bugs

- **Bucket size off-by-one:** need `n + 1` buckets because frequency can be `n`.
- **Wrong initialization:** `[[]] * n` shares one list; use `[[] for _ in range(n)]`.
- **Walking buckets in wrong direction:** want highest frequencies first.
- **Stopping condition:** when `len(res) == k`, return immediately or break out of both loops.

---

## Pattern 7: Sequence Start Detection

### Signal phrases

- "Longest consecutive sequence"
- "Longest run of X"
- "O(n) time, unsorted input"

### When to use

When you need to find sequential structure in an unsorted array, AND the problem requires O(n) time (sorting would be O(n log n)).

### The breakthrough idea

Don't try to grow a sequence from every element — only grow from elements that are **the start of a sequence**. A number `n` is a sequence start if `n - 1` is NOT in the set.

This guarantees that even though there's a nested loop, total work is O(n) because each sequence element is touched at most twice (once as the start-check failure, once as part of the actual walk).

### Template

```python
def longest_consecutive(nums):
    num_set = set(nums)
    longest = 0
    
    for n in nums:
        # Only walk from sequence starts
        if (n - 1) not in num_set:
            length = 1
            while (n + length) in num_set:
                length += 1
            longest = max(longest, length)
    
    return longest
```

### Why it's O(n) and not O(n²)

For each unique element, exactly one of these is true:
- It's a sequence start → we walk forward through its entire sequence ONCE
- It's not a sequence start → we do O(1) work (check `n - 1 in set`) and skip

Total inner-loop work across all sequence starts = sum of sequence lengths = O(n).

### Example problems

**#128 Longest Consecutive Sequence** — the canonical example.

This pattern also generalizes to:
- Find all consecutive subsequences
- Number of distinct sequences in a graph (with BFS/DFS visited tracking analogous to "is this a start")

### Common bugs

- **Counting links instead of elements:** sequence `1→2→3→4` has length 4, not 3.
- **Walking from every element:** that's O(n²). The start-check is non-negotiable.
- **Using a list instead of a set:** `n+1 in lst` is O(n), destroys the complexity.
- **Forgetting to handle empty input:** return 0 immediately.

---

# Auxiliary Pattern: Length-Prefix Encoding

Not really an "Arrays & Hashing" pattern, but appears in this section because of Encode/Decode Strings.

### Signal phrases

- "Serialize a list into a single string"
- "Encode arbitrary strings safely"
- "Parse variable-length records"

### Template

```python
def encode(strs):
    return "".join(f"{len(s)}#{s}" for s in strs)

def decode(s):
    res = []
    i = 0
    while i < len(s):
        # Find the # separating length from content
        j = i
        while s[j] != '#':
            j += 1
        length = int(s[i:j])
        res.append(s[j + 1 : j + 1 + length])
        i = j + 1 + length
    return res
```

### Why length-prefix beats delimiter-only

If you just used a delimiter like `,`, you'd fail when the data itself contains `,`. Length-prefix is delimiter-free for content — you skip exactly `length` characters regardless of what's in them.

### Example problems

**#271 Encode and Decode Strings** — direct application.

### Common bugs

- **Multi-digit length:** if length is 12, `int(s[i+1])` only reads the `1`. Read until you hit `#`.
- **Pointer math:** carefully compute how far to skip after parsing.

---

# Complexity Summary

| # | Problem | Optimal Time | Optimal Space | Pattern(s) |
|---|---------|-------------|---------------|------------|
| 217 | Contains Duplicate | O(n) | O(n) | Set membership |
| 242 | Valid Anagram | O(n) | O(n) | Frequency count |
| 1 | Two Sum | O(n) | O(n) | Complement lookup |
| 49 | Group Anagrams | O(n·k) | O(n·k) | Canonical key grouping |
| 347 | Top K Frequent | O(n) | O(n) | Bucket sort + freq count |
| 271 | Encode/Decode Strings | O(n) | O(n) | Length-prefix encoding |
| 238 | Product Except Self | O(n) | O(1) extra | Prefix + suffix |
| 36 | Valid Sudoku | O(1)\* | O(1)\* | Multi-set membership |
| 128 | Longest Consecutive | O(n) | O(n) | Set + sequence start |

\* Sudoku is technically O(81) = O(1) since the board is fixed-size.

---

# Problem-to-Pattern Map

```text
Pattern 1: Set Membership
├── #217 Contains Duplicate     (direct)
├── #36  Valid Sudoku           (multiple sets)
└── #128 Longest Consecutive    (set for lookup, see Pattern 7)

Pattern 2: Frequency Counting
├── #242 Valid Anagram          (compare two counts)
├── #49  Group Anagrams         (count as canonical key)
└── #347 Top K Frequent         (count first, then bucket)

Pattern 3: Complement Lookup
└── #1   Two Sum                (direct)

Pattern 4: Canonical Key Grouping
└── #49  Group Anagrams         (sorted tuple OR freq tuple as key)

Pattern 5: Prefix + Suffix
└── #238 Product Except Self    (direct)

Pattern 6: Bucket Sort
└── #347 Top K Frequent         (buckets indexed by frequency)

Pattern 7: Sequence Start Detection
└── #128 Longest Consecutive    (only walk from valid starts)

Auxiliary: Length-Prefix Encoding
└── #271 Encode/Decode Strings  (direct)
```

---

# Common Bugs Across the Section

Bugs that show up repeatedly. Internalize these.

### 1. Mutable default vs independent lists

```python
# WRONG — all sublists are the SAME list
buckets = [[]] * 10
buckets[0].append(1)
# buckets is now [[1], [1], [1], ..., [1]]

# RIGHT
buckets = [[] for _ in range(10)]
```

### 2. Using `{x}` when you mean `x`

```python
# WRONG — creates a set containing i
d[key] = {i}

# RIGHT
d[key] = i
```

### 3. `.append()` returns None

```python
# WRONG — assigns None back
d[k] = d[k].append(x)

# RIGHT (mutates in place)
d[k].append(x)
```

### 4. Using a list as a dict key

```python
# WRONG — TypeError: unhashable type: 'list'
d[sorted(s)] = ...

# RIGHT
d[tuple(sorted(s))] = ...
```

### 5. Modifying a set/dict while iterating

```python
# WRONG — RuntimeError: dictionary changed size during iteration
for k in d:
    if condition(k):
        del d[k]

# RIGHT — iterate over a copy of keys
for k in list(d.keys()):
    if condition(k):
        del d[k]
```

### 6. Off-by-one in reverse range

```python
# WRONG — stops at index 1, skips 0
for i in range(n-1, 0, -1):

# RIGHT — includes index 0
for i in range(n-1, -1, -1):
```

### 7. Set destroys order

```python
nums = [1, 100, 2, 200]
list(set(nums))   # might be [200, 1, 2, 100] — order is UNDEFINED
```

If you need order, sort after the set, or use `dict.fromkeys()` (Python 3.7+ preserves insertion order):
```python
unique_ordered = list(dict.fromkeys(nums))
```

### 8. Check-before-add ordering in complement lookup

```python
# WRONG — element matches itself
for i, n in enumerate(nums):
    seen[n] = i
    if target - n in seen:   # could find itself
        return ...

# RIGHT — check first, then add
for i, n in enumerate(nums):
    if target - n in seen:
        return ...
    seen[n] = i
```

### 9. Frequency comparison with zeros

```python
defaultdict(int) comparison can have subtle issues if one dict was
accessed with missing keys (creating zero entries) and the other wasn't.
Safer: build both manually OR use Counter (which doesn't auto-create).
```

### 10. Counting links vs counting elements

```text
1 → 2 → 3 → 4 has 3 LINKS but 4 ELEMENTS

If question asks "length of sequence" → 4
If question asks "number of jumps" → 3
```

---

# Interview Strategy

When you see an Arrays & Hashing problem, follow this script:

### 1. Restate the problem

"So I need to [X] given [Y]. Edge cases include [Z]."

### 2. Brute force first (out loud)

Always articulate the O(n²) brute force. Interviewers want to see this thinking.

> "The brute force would be nested loops, comparing every pair, giving O(n²). The bottleneck is the inner loop — can I avoid re-scanning?"

### 3. Identify the pattern

Walk through the decision tree out loud:
> "I need to check 'have I seen this' / 'what would I need' / 'group by X' → that's [pattern]."

### 4. Choose data structure

> "I'll use a hash set / hash map / array of counts because [reason]."

### 5. Write template, then specialize

Don't write code from scratch. Pattern-match against the template you've memorized, then plug in problem-specific details.

### 6. Walk through example before coding

Pick a small example. Trace it on the whiteboard. THIS catches off-by-ones before you write code.

### 7. State complexity

"This is O(n) time, O(n) space, because [reason]."

### 8. Mention trade-offs

"I'm trading O(n) space for the O(n) time. If the input were sorted, I could use two pointers for O(1) space, but it's not."

---

# Self-Test

Answer these without looking. If any are shaky, re-read that section.

### Conceptual

1. What's the difference between when to use a set vs a hash map?
2. Why is sorting O(n log n) and frequency counting O(n)?
3. Why does the "sequence start detection" pattern give O(n) despite a nested loop?
4. Why must you check BEFORE adding in Two Sum?
5. Why do you need a tuple, not a list, as a dict key?
6. When does bucket sort beat heap sort, and why?
7. Why use length-prefix encoding instead of just a delimiter?

### Pattern recognition (name the pattern)

1. "Find if any element appears more than once" → ?
2. "Find two elements that sum to target" → ?
3. "Group strings by their characters regardless of order" → ?
4. "Compute the product of all elements except current, no division" → ?
5. "Find the 3 most common values" → ?
6. "Find longest run of consecutive integers" → ?
7. "Validate a sudoku board" → ?

### Code

Without looking, can you write the template for each pattern from scratch?

1. Set-based duplicate check
2. Hash map frequency count
3. Two Sum one-pass
4. Group Anagrams
5. Product Except Self with O(1) extra space
6. Top K Frequent with bucket sort
7. Longest Consecutive Sequence

If yes to all → you own this section. Move to Two Pointers.

### Bugs

What's wrong with each of these?

```python
# Bug 1
buckets = [[]] * 10
buckets[0].append(1)
print(buckets)
```

```python
# Bug 2
for i, n in enumerate(nums):
    seen[n] = i
    if target - n in seen:
        return [seen[target - n], i]
```

```python
# Bug 3
d = defaultdict(list)
d[k] = d[k].append(x)
```

```python
# Bug 4
key = sorted(s)
groups[key].append(s)
```

```python
# Bug 5
for i in range(len(nums) - 1, 0, -1):
    process(nums[i])
```

(Answers: 1 — shared list reference; 2 — adds before check, matches self; 3 — `.append()` returns None; 4 — list isn't hashable, need tuple; 5 — skips index 0)

---

# Final Mental Model

Arrays & Hashing is fundamentally about **trading space for time**. You add a hash map or set (O(n) space) to remove a redundant scan (O(n) factor from time).

Every problem in this section asks one of seven questions:

```text
"Have I seen this?"      → Set
"How many of this?"      → Hash Map (freq)
"Where is this?"         → Hash Map (val → index)
"What goes with this?"   → Complement Lookup
"What groups this?"      → Canonical Key
"All-but-this aggregate?"→ Prefix + Suffix
"Bounded sort?"          → Bucket Sort
"Sequential structure?"  → Set + Start Detection
```

If you can map a problem to one of these in under 30 seconds, the implementation is mechanical.

That mapping is the skill. The code is the rest.

---

# Where to Go Next

After this section is locked in:

1. **Two Pointers** — reuses set/dict techniques but adds pointer movement
2. **Sliding Window** — frequency counting (Pattern 2) inside a moving window
3. **Stack** — pattern-recognition shifts but problem-solving discipline is the same
4. **Trees + BFS/DFS** — visited sets, level counts, all reuse Pattern 1 and 2

Every section above this one assumes you can fluently apply these 7 patterns. Master here = leverage everywhere.
