# 704. Binary Search

## Problem
Given a sorted (ascending) array of distinct integers and a target, return the index of the target, or `-1` if it's not there. Must run in `O(log n)`.

Example: `nums = [-1,0,3,5,9,12]`, `target = 9` → `4`.

## Learning Journey
Came in with a working skeleton and two real bugs. First fix was clean — caught the operator-precedence issue immediately and corrected `l+r // 2` to `(l+r) // 2`. For the second bug (the loop missing the final candidate), instead of switching the loop condition to `l <= r`, took the alternate route: kept `while l < r` and added a post-loop `if nums[r] == target` check.

That alternate structure is correct — verified by tracing several inputs (found target at start/middle/end, target absent, target below range). The post-loop check works because the sorted invariant guarantees `nums[r]` at exit is either the one leftover candidate (`l == r`) or an already-ruled-out element that can't false-positive.

Note for honesty: tutor briefly hinted toward a bug in the post-loop check that doesn't actually exist. There isn't one — the sorted-order argument closes it.

## Attempts

**Attempt 1 — two bugs**
```python
class Solution:
    def search(self, nums: List[int], target: int) -> int:
        l, r = 0, len(nums) - 1
        while l < r:
            mid = l + r // 2
            if nums[mid] == target:
                return mid
            elif nums[mid] > target:
                r = mid - 1
            else:
                l = mid + 1
        return -1
```

| Bug | Type | Explanation |
|-----|------|-------------|
| `mid = l + r // 2` | Implementation (precedence) | `//` binds tighter than `+`, so this is `l + (r // 2)`, not `(l + r) // 2`. Identical while `l == 0`, wrong the moment `l` moves — overshoots past the array. |
| `while l < r` with no leftover check | Edge Case | Inclusive bounds (`r=mid-1`, `l=mid+1`) mean `l`/`r` always sit on unchecked candidates. When they collide on the final candidate, `l < r` exits before checking it. |

Breaking trace for bug 1 — `nums=[1,2,3,4,5]`, `target=5`:

| step | l | r | `l + r//2` | nums[mid] | action |
|------|---|---|------------|-----------|--------|
| 1 | 0 | 4 | `0+2 = 2` | 3 | 3<5, l=3 |
| 2 | 3 | 4 | `3+2 = 5` | nums[5] | **IndexError** |

Breaking input for bug 2 — `nums=[5]`, `target=5`: loop condition `0 < 0` false, never enters, returns `-1` instead of `0`.

**Attempt 2 — accepted**
```python
class Solution:
    def search(self, nums: List[int], target: int) -> int:
        l, r = 0, len(nums) - 1
        while l < r:
            mid = (l + r) // 2
            if nums[mid] == target:
                return mid
            elif nums[mid] > target:
                r = mid - 1
            else:
                l = mid + 1
        if nums[r] == target:
            return r
        return -1
```

## Accepted Solution
```python
class Solution:
    def search(self, nums: List[int], target: int) -> int:
        l, r = 0, len(nums) - 1
        while l < r:
            mid = (l + r) // 2
            if nums[mid] == target:
                return mid
            elif nums[mid] > target:
                r = mid - 1
            else:
                l = mid + 1
        if nums[r] == target:
            return r
        return -1
```
Result: Accepted. _(Fill in runtime/percentile — LC binary search runtime % is noisy, don't read into it.)_

## Why It Works
**Invariant:** if `target` is in the array, it's always within `[l, r]`. Every branch shrinks the range while preserving that — `nums[mid] > target` rules out `mid` and everything right (`r=mid-1`), `nums[mid] < target` rules out `mid` and everything left (`l=mid+1`).

**Key operation:** halving the search range each step via `mid`, then discarding the half that can't contain the target.

**Load-bearing line:** `if nums[r] == target` after the loop. Because `while l < r` exits one element early, this is what checks the final surviving candidate. It's safe even when `r` stepped below `l`, because sorted order means that element is already ruled out and can't equal the target.

## How It Works
**Traversal:** two pointers `l` and `r` bound the live range. Each iteration computes `mid` at the floor-midpoint and jumps one pointer to the far side of `mid`, never the slow one-step march — that's what makes it `log n`.

```
[-1, 0, 3, 5, 9, 12]   target 9
  l        m        r        mid=2, 3<9 -> l=mid+1
           l     m  r        mid=4, 9==9 -> return 4
```

**Business:** the decision at each step —
1. Compute `mid` = `(l+r)//2`.
2. If `nums[mid] == target`: found it, return `mid`.
3. If `nums[mid] > target`: target is left, drop `mid` and right → `r = mid-1`.
4. Else: target is right, drop `mid` and left → `l = mid+1`.
5. After loop: check the lone leftover at `nums[r]`.

**Variables:**
| Variable | Represents | Stores | How It Changes | Why It Exists |
|----------|------------|--------|----------------|---------------|
| `l` | left bound of live range | index | only increases, `l = mid+1` | marks lowest index target could still be at |
| `r` | right bound of live range | index | only decreases, `r = mid-1` | marks highest index target could still be at |
| `mid` | current probe point | index | recomputed each loop as `(l+r)//2` | the element we test to decide which half to keep |

## Visual Walkthrough
`nums = [-1, 0, 3, 5, 9, 12]`, `target = 9`:

| Step | l | r | mid | nums[mid] | Action | Result |
|------|---|---|-----|-----------|--------|--------|
| 1 | 0 | 5 | 2 | 3 | 3 < 9, go right | l = 3 |
| 2 | 3 | 5 | 4 | 9 | 9 == 9 | return 4 |

Leftover-check path — `nums = [2, 5]`, `target = 5`:

| Step | l | r | mid | nums[mid] | Action | Result |
|------|---|---|-----|-----------|--------|--------|
| 1 | 0 | 1 | 0 | 2 | 2 < 5, go right | l = 1 |
| exit | 1 | 1 | — | — | `1 < 1` false | post-loop `nums[1]==5` → return 1 |

## Optimization Journey
1. **Brute force:** scan left to right, return first index where `nums[i] == target`. `O(n)`.
2. **Bottleneck:** linear scan ignores that the array is sorted — it re-checks elements that the ordering already rules out.
3. **Observation:** comparing target to the middle element tells you which half the answer must be in. Half the array is eliminated per comparison.
4. **Optimization:** binary search — probe the midpoint, discard the impossible half, repeat.
5. **Final:** `O(log n)`. An interviewer reaches this the moment they ask "what does *sorted* buy us that a scan throws away?"

## Pattern Recognition
**Signals:**
- "sorted array" + "find target" → binary search, almost reflexively.
- explicit `O(log n)` requirement → binary search is basically the only fit.

**Pattern used:** Binary Search — the array is sorted and we're locating a value, so each comparison halves the space.

| Pattern | Why It Fits | Why It Doesn't | Verdict |
|---------|-------------|----------------|---------|
| Binary Search | sorted + find index, `O(log n)` | — | ✅ correct |
| Hash Set / Map | gives `O(1)` membership | builds in `O(n)` space/time, throws away the sorted property the problem hands you | ❌ red herring |
| Two Pointers | `l`/`r` are pointers | it's the *mechanism* of binary search, not a separate strategy | n/a |

## Edge Cases & Debugging
| Case | Input | Expected | How Handled |
|------|-------|----------|-------------|
| single element, is target | `[5]`, t=5 | 0 | post-loop `nums[r]` check catches it |
| target at last index | `[2,5]`, t=5 | 1 | loop collides at `r=1`, post-loop returns it |
| target absent, below range | `[-1,0,3]`, t=-5 | -1 | `r` walks to -1, `nums[-1]` ≠ target (sorted invariant), returns -1 |
| target at first index | `[-1,0,3]`, t=-1 | 0 | found mid-loop |
| empty array (latent) | `[]` | n/a (LC guarantees len≥1) | would `IndexError` on `nums[-1]` — footgun of the post-loop-index style, not triggered under LC 704 constraints |

## Complexity
| Metric | Complexity | Reason |
|--------|------------|--------|
| Time   | `O(log n)` | range halves every iteration |
| Space  | `O(1)` | two index pointers, iterative — no recursion stack |

## Interview Prep
**30s:** Sorted array plus find-target plus `O(log n)` means binary search. Two pointers bound the live range, probe the midpoint, discard the half that can't hold the target. Log-n time, constant space.

**2 min:** The recognition signal is "sorted" — that's what lets each comparison eliminate half the array instead of one element. Invariant: if the target exists, it's always inside `[l, r]`. Key operation: compare `nums[mid]` to target, move `l` or `r` past `mid` accordingly. The one subtlety in this implementation is the `while l < r` loop exits one element early, so a post-loop `nums[r]` check handles the final candidate. Tradeoff vs the `while l <= r` style: this version does one comparison after the loop instead of inside it — same complexity, slightly different bookkeeping, and it carries an empty-array footgun.

**Follow-ups they'll ask:**
- *"What if there are duplicates and you want the first occurrence?"* → lower-bound variant: on `nums[mid] == target`, don't return — record and keep searching left (`r = mid - 1`).
- *"Rotated sorted array?"* → still binary search, but first decide which half is sorted, then check if target falls in that half.
- *"Why `(l+r)//2` and not something else?"* → in languages with fixed-width ints, `l+r` can overflow; `l + (r-l)//2` avoids it. Python ints are arbitrary precision so it's moot here, but interviewers like hearing you know.

## Revision
**1 week:** re-state the invariant ("target, if present, stays in `[l,r]`") and re-trace the breaking input `nums=[5], target=5` — the one that exposed the missing leftover check.

**1 month:** one-line skeleton — `while l<r: mid=(l+r)//2; move l or r past mid`, then post-loop check `nums[r]`. Complexity `O(log n)` because the range halves each step.

**Mental model:** looking up a word in a physical dictionary. You don't read page by page — you flip to the middle, see if your word's before or after, and throw away half the book. Repeat until one page is left, then check that page. The post-loop check is "actually look at the final page you landed on."

**Takeaways:**
- Operator precedence: `//` binds tighter than `+`. Always parenthesize `(l+r)//2`. _(recurring bug #1)_
- Two valid loop styles: `while l <= r` (check inside) vs `while l < r` + post-loop check. Pick one and know why it terminates. _(recurring bug #2)_
- Post-loop index checks have an empty-input footgun — fine under LC constraints, dangerous in general code.

## Recurring Bug Tracker (start of list)
```
#1 operator precedence — wrote l+r // 2 instead of (l+r) // 2
#2 inclusive-bounds loop missing the final candidate (while l<r with no leftover check)
```
