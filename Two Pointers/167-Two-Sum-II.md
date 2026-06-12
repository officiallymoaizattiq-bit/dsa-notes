# 167. Two Sum II - Input Array Is Sorted

## Problem

Given a **1-indexed**, sorted-ascending array `numbers` and a `target`, return the indices `[index1, index2]` of the two numbers that add up to `target`, where `1 <= index1 < index2 <= numbers.length`. Exactly one solution exists, and you can't reuse the same element.

Concrete example:

```
Input:  numbers = [2, 7, 11, 15], target = 9
        2 + 7 = 9
Output: [1, 2]   (1-indexed positions of 2 and 7)
```

---

# My Learning Journey

i recognized two pointers immediately — the array being sorted is the giant tell — so the pattern was never in question. the whole learning curve on this one was about *how* the pointers move, which is the key difference from problem 125.

my first attempt had the right skeleton (`l, r` converging) but the movement was broken. i was only ever decrementing `r`, with a weird `if l == r-1: l += 1` hack trying to limp the left pointer forward. that completely threw away the advantage of the array being sorted. the realization that unlocked it: **the sum tells you which direction to move.** too big → shrink it. too small → grow it. once that clicked, the hack disappeared and the three-way logic fell out naturally.

second attempt got the direction right but used three independent `if`s, which meant a single iteration could move both pointers and compare a pair i never cleanly evaluated. the fix was `if/elif/else` so exactly one decision happens per pass. i also kept forgetting the return value is **1-indexed** — that took two reminders before `[l+1, r+1]` stuck.

the deeper thing i took away: this is the *same* two-pointer skeleton as valid palindrome, but the movement is **data-driven** here (a comparison decides who moves) versus **unconditional** there (both always step inward). same shape, different engine.

---

# Attempt 1

```python
class Solution:
    def twoSum(self, numbers: List[int], target: int) -> List[int]:
        l, r = 0, len(numbers) - 1
        while l < r:
            if numbers[l] + numbers[r] == target:
                return [l, r]
            r -= 1
            if l == r - 1:
                l += 1
```

**what i was thinking:** two pointers, check for the target, shuffle the pointers somehow.

**what went wrong — logic bug in pointer movement:** i only ever moved `r` down, with a hack to occasionally bump `l`. that ignores the sorted structure entirely. the sum should *drive* the direction — i wasn't using it. also returned 0-indexed `[l, r]` instead of 1-indexed.

---

# Attempt 2

```python
class Solution:
    def twoSum(self, numbers: List[int], target: int) -> List[int]:
        l, r = 0, len(numbers) - 1
        while l < r:
            if numbers[l] + numbers[r] < target:
                l += 1
            if numbers[l] + numbers[r] > target:
                r -= 1
            if numbers[l] + numbers[r] == target:
                return [l, r]
```

**what i fixed:** got the directional logic — too small move `l`, too big move `r`. dropped the `l == r-1` hack.

**what was still wrong:**
- three independent `if`s instead of `if/elif`. a single iteration could move `l`, then re-check and move `r`, comparing a pair that was never cleanly evaluated. each iteration must make exactly *one* decision.
- the `== target` check ran *last*, on already-moved pointers, instead of first on the current pair.
- still returning 0-indexed.

---

# Attempt 3 (Accepted)

```python
class Solution:
    def twoSum(self, numbers: List[int], target: int) -> List[int]:
        l, r = 0, len(numbers) - 1
        while l < r:
            if numbers[l] + numbers[r] < target:
                l += 1
            elif numbers[l] + numbers[r] > target:
                r -= 1
            else:
                return [l + 1, r + 1]
```

**what i fixed:** `if/elif/else` so exactly one branch fires per iteration. the `else` is safe because if it's not `<` and not `>`, it must be `==`. and `[l+1, r+1]` handles the 1-indexing.

---

# Accepted Solution(s)

```python
class Solution:
    def twoSum(self, numbers: List[int], target: int) -> List[int]:
        l, r = 0, len(numbers) - 1
        while l < r:
            if numbers[l] + numbers[r] < target:   # sum too small → need bigger → move l up
                l += 1
            elif numbers[l] + numbers[r] > target: # sum too big → need smaller → move r down
                r -= 1
            else:                                  # exactly equal → found it
                return [l + 1, r + 1]              # 1-indexed return
```

---

# Why It Works

**The invariant:** because the array is sorted ascending, the sum at `(l, r)` tells you which direction the answer must be in. If the sum is too big, the only way to shrink it is to drop `r` — moving `l` right would make it bigger. If too small, the only way to grow it is to advance `l`. So every move provably eliminates candidates that *cannot* contain the answer. The search space shrinks by one full row or column each step, which is what turns O(n²) into O(n).

---

# Traversal Logic

How we move through the data — two pointers converging, but **direction is data-driven**.

```
numbers = [2, 7, 11, 15], target = 9

 [ 2,  7, 11, 15 ]
   ^           ^
   l           r        sum = 17 > 9 → move r left

 [ 2,  7, 11, 15 ]
   ^       ^
   l       r            sum = 13 > 9 → move r left

 [ 2,  7, 11, 15 ]
   ^   ^
   l   r                sum = 9 == 9 → return
```

Unlike valid palindrome (where both pointers always step inward unconditionally), here **the comparison decides who moves**. Only one pointer moves per iteration.

---

# Business Logic

The decision made at each step — separate from how we move.

- compute the current sum
- `== target` → found the pair, return 1-indexed
- `< target` → sum too small, need a bigger value → advance `l`
- `> target` → sum too big, need a smaller value → retreat `r`

The *comparison* is business. The *resulting pointer move* is traversal. The link between them — sum drives direction — is the heart of this problem.

---

# Variable Roles

| Variable | Represents | Stores | How It Changes | Why It Exists |
|----------|------------|--------|----------------|---------------|
| `l` | left pointer | int index | `+= 1` when sum too small | front of search window; advancing it raises the sum |
| `r` | right pointer | int index | `-= 1` when sum too big | back of search window; retreating it lowers the sum |
| `target` | the goal sum | int | never changes | the value we're trying to hit |

---

# Visual Walkthrough

Input: `numbers = [2, 7, 11, 15]`, `target = 9`

| Step | l | r | numbers[l] | numbers[r] | Sum | vs Target | Action |
|------|---|---|-----------|-----------|-----|-----------|--------|
| 1 | 0 | 3 | 2 | 15 | 17 | > 9 | r → 2 |
| 2 | 0 | 2 | 2 | 11 | 13 | > 9 | r → 1 |
| 3 | 0 | 1 | 2 | 7 | 9 | == 9 | return [1, 2] |

---

# Optimization Journey

1. **Brute Force:** nested loop checking every pair. O(n²) time, O(1) space.
2. **Bottleneck:** the O(n²) — re-examining pairs that the sorted order already rules out.
3. **Observation:** the array is *sorted*. At any `(l, r)`, the sum tells you which direction the answer must lie, so you can discard a whole row/column per step.
4. **Optimization:** converging two pointers, moving the one that pushes the sum toward target.
5. **Final Approach:** the accepted solution — O(n) time, O(1) space. An interviewer discovers this by asking "what does the sorted property let me skip?"

---

# Pattern Recognition

## Signal Phrases

- "sorted array" → enables directional two-pointer movement
- "two numbers that add up to target" → pair-sum problem
- "1-indexed" → watch the return offset
- implicit O(1) space expectation given the sorted input

## Pattern Used

Two Pointers (converging, data-driven movement).

## Why This Pattern Fits

- array is sorted, so the sum at any pair tells you which way to move
- looking for a single pair summing to target
- achieves O(n) time, O(1) space — strictly better than the hash map here

---

# Pattern Comparison

| Pattern | Why It Fits | Why It Doesn't | Verdict |
|---------|-------------|----------------|---------|
| Two Pointers | sorted array → sum drives direction, O(1) space | — | ✅ optimal |
| Hash Map | classic two sum, O(n) time | O(n) space, ignores the sorted gift | ⚠️ works, wasteful here |
| Binary Search (per element) | find complement of each `numbers[i]` | O(n log n), slower than two pointers | ❌ worse |
| Brute Force | trivially correct | O(n²) | ❌ too slow |

---

# Common Mistakes

These are the ones I actually hit:

- only moving one pointer (`r`) and hacking the other forward with `if l == r-1` — ignores the sorted structure
- using three independent `if`s instead of `if/elif/else`, letting both pointers move in one iteration
- checking `== target` *after* moving the pointers instead of on the current pair
- returning 0-indexed `[l, r]` instead of 1-indexed `[l+1, r+1]`

---

# Edge Cases

| Case | Input | Expected | How Handled |
|------|-------|----------|-------------|
| answer at the ends | `[2, 7]`, t=9 | [1, 2] | first compare hits `==` |
| answer in the middle | `[1, 2, 3, 4]`, t=5 | [2, 3] | pointers converge inward via sum direction |
| sum starts too high | `[2, 7, 11, 15]`, t=9 | [1, 2] | `r` walks down until match |
| negative numbers | `[-3, 0, 3]`, t=0 | [1, 3] | same logic, sum comparison still valid |

---

# Debugging Lessons

The real lesson: **on a sorted array, let the comparison choose the pointer.** My instinct was to move pointers on a fixed schedule (always `r`, occasionally `l`), which is the valid-palindrome reflex. But here the movement is *conditional* — the sum vs target decides. Recognizing that two-pointer problems split into "unconditional convergence" vs "data-driven convergence" is the takeaway.

Secondary: `if/elif/else` vs stacked `if`s genuinely changes behavior when the branches mutate the state the next branch reads. Use `elif` whenever exactly one action should happen per iteration.

---

# NeetCode / Official Solution

## Code

```python
class Solution:
    def twoSum(self, numbers: List[int], target: int) -> List[int]:
        l, r = 0, len(numbers) - 1
        while l < r:
            curSum = numbers[l] + numbers[r]
            if curSum > target:
                r -= 1
            elif curSum < target:
                l += 1
            else:
                return [l + 1, r + 1]
```

## Comparison to My Solution

| | NeetCode | Mine |
|---|----------|------|
| Algorithm | two pointers, data-driven | two pointers, data-driven |
| Sum computed | once into `curSum` | recomputed in each branch check |
| Branch order | `> target` first | `< target` first (irrelevant — mutually exclusive) |
| Time / Space | O(n) / O(1) | O(n) / O(1) |
| 1-indexing | `[l+1, r+1]` | `[l+1, r+1]` |

Functionally identical. The only meaningful difference: NeetCode stores the sum once in `curSum` instead of recomputing `numbers[l] + numbers[r]` per branch. Zero performance impact (it's the same O(1) op), but cleaner to read and the version to write on a whiteboard.

---

# Deep Understanding

The non-obvious part is *why* moving one pointer safely discards candidates. Picture the full grid of all pair-sums, rows indexed by `l` and columns by `r`, sorted both ways. You start in one corner. When `sum > target`, every pair using the current `r` with an even larger `l` would be *even bigger* — so that entire column is dead, and dropping `r` discards it in one move. Same logic mirrored for `sum < target` and the `l` rows. Each step eliminates a whole line of the grid, so you touch at most `n` cells total → O(n). That's the geometric reason two pointers is correct, not just convenient.

---

# Complexity Analysis

| Metric | Complexity | Reason |
|--------|------------|--------|
| Time | O(n) | `l` only increases, `r` only decreases; combined ≤ n moves before crossing |
| Space | O(1) | two int pointers, nothing allocated — the payoff of the sorted input |

---

# Related Problems

| # | Problem | Difficulty | Same Pattern? | Notes |
|---|---------|------------|---------------|-------|
| 1 | Two Sum | Easy | No (hash map) | unsorted input → two pointers doesn't apply directly |
| 15 | 3Sum | Medium | Yes (builds on this) | sort, fix one element, two-pointer the rest |
| 125 | Valid Palindrome | Easy | Yes (two pointers) | but movement is *unconditional* there, not data-driven |
| 11 | Container With Most Water | Medium | Yes | move the smaller-height pointer |

---

# Interview Explanation (30 Seconds)

Since the array is sorted, I use converging two pointers. I start at both ends and look at the sum: too big, I move the right pointer down; too small, I move the left pointer up; equal, I've found it. The sorted property guarantees each move only discards pairs that can't be the answer. O(n) time, O(1) space.

---

# Interview Explanation (2 Minutes)

The signals are "sorted array" plus "find a pair summing to target" — that points at converging two pointers rather than a hash map, because the sorted order lets the sum tell me which direction to move.

I place `l` at the start and `r` at the end. Each iteration I compute the sum. If it equals target, I return the indices (1-indexed, so `[l+1, r+1]`). If the sum is too big, the only way to reduce it is to move `r` left toward smaller values. If too small, I move `l` right toward bigger values. Exactly one pointer moves per iteration, driven by the comparison.

This is O(n) because each pointer only travels in one direction and they never reset — combined they move at most n steps. It's O(1) space, which is the advantage over the hash-map two-sum that needs O(n). The correctness argument is that each move discards an entire set of pairs that the sorted order guarantees can't reach the target. The main tradeoff to mention: if the array weren't sorted, I'd fall back to the hash map.

---

# Common Follow-Up Questions

- **"What if the array weren't sorted?"** → either sort first (O(n log n)) then two-pointer, or use the hash-map two-sum (O(n) time, O(n) space).
- **"What if there are multiple valid pairs?"** → adapt to collect all: on a match, record it, then move *both* pointers and continue (and skip duplicates if needed — that's the 3Sum technique).
- **"Why is this O(n) and not O(n²)?"** → pointers are monotonic and never reset; total movement bounded by n.
- **"How does this extend to 3Sum?"** → sort, fix the first element, run this exact two-pointer scan on the remainder for each fixed element → O(n²).

---

# Key Takeaways

- sorted array + pair-sum → converging two pointers, **let the sum drive the direction**.
- this is the data-driven cousin of valid palindrome's unconditional convergence — same skeleton, different movement rule.
- use `if/elif/else` when exactly one action should happen per iteration.
- mind the indexing convention — this problem is 1-indexed.

---

# Revision Notes

When re-reading in a week/month, the minimum to refresh:
- skeleton: `l, r` at the ends, compare sum to target, move the pointer that pushes sum toward target, `==` returns.
- the one insight: sorted-ness means the sum tells you which way to go.
- the one gotcha: 1-indexed return `[l+1, r+1]`.
- the contrast to hold onto: data-driven movement (167) vs unconditional movement (125).

---

# Final Mental Model

Think of two people standing at opposite ends of a number line of sorted values, trying to make their two numbers add up to a target. If their total is too high, the person on the big end steps inward to a smaller number. If it's too low, the person on the small end steps inward to a bigger one. They keep stepping until they either meet or hit the target. The sorted line is what makes each step a guaranteed-correct guess about which way to go.
