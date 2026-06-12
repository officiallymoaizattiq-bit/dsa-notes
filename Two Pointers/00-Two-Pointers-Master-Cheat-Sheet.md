# Two Pointers — Master Cheat Sheet

Read this before solving any two-pointer problem. Distilled from 125, 167, 15, 11, 42.

---

# 1. When Is It Two Pointers?

Reach for two pointers when you see **any** of these signals:

| Signal | Example |
|--------|---------|
| "palindrome" / symmetry / reads same both ways | 125 |
| **sorted** array + find a pair/triplet summing to a target | 167, 15 |
| pick two elements to maximize/minimize something between them | 11 |
| water / area / span bounded by two sides | 11, 42 |
| "in-place" or O(1) space expected on an array problem | all |
| you'd otherwise check all O(n²) pairs — can you prune? | 11, 167 |

**The unlock question:** *"There are O(n²) pairs. Can I move a pointer in a way that provably skips pairs that can't be the answer?"* If yes → two pointers, O(n).

**If the array isn't sorted but two pointers would help → sort it first** (15). The sort cost (O(n log n)) is usually dominated by the rest. BUT — only sort if order doesn't matter to the answer. If you need original indices, sorting breaks that (that's why plain Two Sum uses a hash map, not two pointers).

---

# 2. The Four Movement Flavors (the meta-skill)

Every two-pointer problem is one of these. Identifying *which* tells you how the pointers move. This is the single most useful thing to recognize.

| Flavor | Movement rule | Problems |
|--------|--------------|----------|
| **Unconditional** | both always step inward every iteration | 125 Valid Palindrome |
| **Sum-driven** | compare sum to target; sum too small → move left up, too big → move right down | 167, 15 |
| **Greedy min-of-two** | move the pointer at the *worse* (shorter/smaller) side; it's the bottleneck | 11 Container |
| **Smaller-running-max** | track a running max each side; resolve the side whose max is smaller | 42 Trapping Rain Water |

**The shared idea:** in 3 of 4 flavors, *the data decides which pointer moves.* Only palindrome moves both blindly. Figure out the decision rule and the code writes itself.

---

# 3. The Skeleton

Converging two pointers (the 90% case):

```python
l, r = 0, len(arr) - 1
while l < r:
    # 1. compute something from arr[l], arr[r]
    # 2. update your answer (running max/min, append, return)
    # 3. move one or both pointers based on the decision rule
```

Outer-loop + inner two-pointer (for triplets, 15):

```python
arr.sort()
for i in range(len(arr)):
    if i > 0 and arr[i] == arr[i-1]:   # skip duplicate anchor
        continue
    l, r = i + 1, len(arr) - 1          # l starts AFTER i
    while l < r:
        ...                             # standard 2-sum scan on the rest
```

Three-pass precompute (42, the O(n)-space version):

```python
# pass 1: build left-info array (left to right)
# pass 2: build right-info array (right to left)
# pass 3: combine per-position
```

---

# 4. Syntax Gotchas (the bugs that bit me repeatedly)

These are the recurring ones across all 5 problems. Check yourself against every one.

### `if/elif/else` vs stacked `if`
When exactly **one** of several actions should happen per iteration, use `if/elif/else`. Stacked `if`s let multiple branches fire in one pass, mutating state the next branch reads.
```python
# WRONG — both can fire, second reads moved pointer
if sum < target: l += 1
if sum > target: r -= 1
# RIGHT — exactly one fires
if sum < target: l += 1
elif sum > target: r -= 1
else: ...
```
*(bit me in 167, 15, 11)*

### Methods that mutate in place return `None`
`.sort()` and `.lower()` (and `.reverse()`, `.append()`) mutate or return — know which.
```python
nums = nums.sort()    # WRONG → nums is now None
nums.sort()           # RIGHT → sorts in place, keep using nums
new = sorted(nums)    # RIGHT → returns a new sorted list
s = s.lower()         # RIGHT → strings are immutable, MUST capture
```
*(bit me in 125 with `s.lower()`, in 15 with `nums.sort()`)*

### Methods are called ON the object, not as free functions
```python
isalnum(s[i])      # WRONG → NameError
s[i].isalnum()     # RIGHT → method on the string
```
*(bit me in 125)*

### `max()`/`min()` need the keyword — `(a, b)` alone is a tuple
```python
maxL = (maxL, height[i])        # WRONG → builds a tuple
maxL = max(maxL, height[i])     # RIGHT
```
*(bit me in 42)*

### Bounds check BEFORE index access in compound conditions
Short-circuit evaluation only protects you if the guard is on the **left**.
```python
while not s[l].isalnum() and l < len(s):   # WRONG → s[l] accessed before guard → IndexError
while l < r and not s[l].isalnum():        # RIGHT → l < r stops eval before s[l]
```
*(bit me in 125)*

### List pre-sizing: `[0] * n`, not `[] * n`
```python
maxRight = [] * len(arr)      # WRONG → still empty, maxRight[i] = ... → IndexError
maxRight = [0] * len(arr)     # RIGHT → [0,0,0,...], indexable
```
*(bit me in 42)*

### Indices vs values — don't confuse `min(l, r)` with `min(arr[l], arr[r])`
`l` and `r` are *positions*. The thing you usually want is the *value* at those positions.
```python
min(l, r)                    # the smaller INDEX — almost never what you want
min(arr[l], arr[r])          # the smaller VALUE — usually what you want
```
*(bit me in 11, 42)*

### You can't advance a `for ... in range()` loop by mutating its variable
The range reassigns the loop variable every iteration. To skip → `continue`. To stop → `break`.
```python
for i in range(n):
    if dup: i += 1     # WRONG → does nothing, range overwrites i next pass, body still runs
    if dup: continue   # RIGHT → skips the rest of this iteration
```
*(bit me in 15)*

### Case sensitivity
`maxL` and `MaxL` are different variables. Pick one casing, use it everywhere.
*(bit me in 42)*

---

# 5. Per-Problem Quick Reference

### 125 Valid Palindrome — *unconditional convergence*
```python
l, r = 0, len(s) - 1
s = s.lower()
while l < r:
    while l < r and not s[l].isalnum(): l += 1   # skip junk, bounds-first
    while l < r and not s[r].isalnum(): r -= 1
    if s[l] != s[r]: return False
    l += 1; r -= 1
return True
```
**Key:** skip non-alphanumeric, compare case-insensitive, both pointers step inward.
**O(n) time, O(1) space** (if lowercasing inline instead of copying).

### 167 Two Sum II — *sum-driven*
```python
l, r = 0, len(numbers) - 1
while l < r:
    s = numbers[l] + numbers[r]
    if s < target: l += 1
    elif s > target: r -= 1
    else: return [l + 1, r + 1]     # 1-INDEXED
```
**Key:** sorted array → sum tells you direction. Too small move l, too big move r. Watch the 1-indexing.
**O(n) time, O(1) space.**

### 15 3Sum — *sort + scan + dedup*
```python
nums.sort()
res = []
for i in range(len(nums)):
    if i > 0 and nums[i] == nums[i-1]: continue   # dedup anchor
    l, r = i + 1, len(nums) - 1
    while l < r:
        s = nums[i] + nums[l] + nums[r]
        if s > 0: r -= 1
        elif s < 0: l += 1
        else:
            res.append([nums[i], nums[l], nums[r]])
            l += 1; r -= 1
            while l < r and nums[l] == nums[l-1]: l += 1   # dedup left
return res
```
**Key:** = "Two Sum II run n times." Fix one, 2-sum the rest against `-nums[i]`. **Dedup the two FREE dials (i and l); r is forced, no dedup needed.** Return values not indices.
**O(n²) time, O(1) space.**

### 11 Container With Most Water — *greedy min-of-two*
```python
res = 0
l, r = 0, len(heights) - 1
while l < r:
    res = max(res, (r - l) * min(heights[l], heights[r]))
    if heights[l] < heights[r]: l += 1   # move the SHORTER wall
    else: r -= 1
return res
```
**Key:** area = width × shorter wall. **Move the shorter wall** — it's the only move with upside (taller-wall move shrinks width while still capped by the same short wall = strictly worse).
**O(n) time, O(1) space.**

### 42 Trapping Rain Water — *smaller-running-max* (optimal)
```python
res = 0
l, r = 0, len(height) - 1
maxL, maxR = height[l], height[r]
while l < r:
    if maxL < maxR:
        l += 1
        maxL = max(maxL, height[l])     # update FIRST → no clamp
        res += maxL - height[l]
    else:
        r -= 1
        maxR = max(maxR, height[r])
        res += maxR - height[r]
return res
```
**Key:** water per position = `min(maxLeft, maxRight) - height[i]`. Process the **smaller-max side** — it's the binding constraint; the taller side can only grow so it never lowers the water line. Update max before subtracting to avoid negative water.
**O(n) time, O(1) space.** (O(n)-space version: precompute maxLeft[] and maxRight[] arrays.)

---

# 6. Complexity Cheat

| Problem | Time | Space | Why O(n) not O(n²) |
|---------|------|-------|--------------------|
| 125 | O(n) | O(1) | pointers monotonic, never reset |
| 167 | O(n) | O(1) | same |
| 15 | O(n²) | O(1) | n anchors × O(n) inner scan |
| 11 | O(n) | O(1) | each move discards a provably-worse wall |
| 42 | O(n) | O(1) | single pass, two scalar maxes |

**The universal reason two pointers is O(n):** the pointers only move toward each other and **never reset**, so combined movement is bounded by n. Nested `while` loops inside the main loop are still O(n) total as long as no pointer resets — they look O(n²) but aren't.

---

# 7. Pre-Solve Checklist

Before writing code on a new two-pointer problem, answer these:

1. **Is the array sorted?** If not and order doesn't matter → sort. If you need original indices → reconsider (maybe hash map).
2. **Which movement flavor?** Unconditional / sum-driven / greedy-min / running-max. This is your decision rule.
3. **Where do pointers start?** Both ends (converging) is default. For triplets, `l = i + 1`.
4. **What's the decision that moves a pointer?** Write it as `if/elif/else`, one branch fires.
5. **Do I compute the answer before or after moving?** Usually compute on the current pair FIRST, then move.
6. **Duplicates?** If "unique" results are required and array is sorted → dedup by skipping adjacent equal values. Dedup only the *free* positions.
7. **Running value or full array?** If you only ever need the current value, use a scalar, not an array (this is the O(n)→O(1) space collapse).
8. **Index vs value?** When you write `min`/`max`/comparisons, am I using `arr[l]` (value) where I mean value, not `l` (index)?

---

# 8. The One-Liners to Recall Cold

- **125:** two people read from both ends inward, skip junk, compare — both step every time.
- **167:** sorted line, the sum tells you which end to pull inward.
- **15:** anchor one number, 2-sum the rest; dedup the dials you turn freely, the forced one is automatic.
- **11:** retire the shorter wall — it's the bottleneck and it's already maxed.
- **42:** walk inward from both ends; stand on the shorter-tallest-wall side because that's where the water depth is already known.

---

# 9. Recurring Bugs — Final Watchlist

Tape this to your monitor:

- [ ] `if/elif/else` when one decision per iteration (not stacked `if`)
- [ ] `.sort()`/`.lower()` — mutate-in-place vs return-new (capture or don't)
- [ ] `arr[i].method()` not `method(arr[i])`
- [ ] `max(a, b)` not `(a, b)` (tuple)
- [ ] bounds check before index access in `and` conditions
- [ ] `[0] * n` not `[] * n`
- [ ] `min(arr[l], arr[r])` not `min(l, r)` — values not indices
- [ ] `continue`/`break` to control a for-loop, never `i += 1`
- [ ] case-sensitive variable names — one casing
- [ ] compute answer on current pair before moving pointers
- [ ] 1-indexed vs 0-indexed return (167!)
