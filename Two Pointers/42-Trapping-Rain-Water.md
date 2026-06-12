# 42. Trapping Rain Water

## Problem

Given a non-negative integer array `height` representing an elevation map where each bar has width 1, return the total units of water trapped after raining.

Concrete example:

```
height = [0, 1, 0, 2, 1, 0, 1, 3, 2, 1, 2, 1]
Output: 6
```

```
height = [0, 1, 0, 2]
Output: 1
```

Unlike Container With Most Water, the **middle bars matter** — every position can trap water on top of itself, and you sum the water across the entire structure (not one max pool).

---

# My Learning Journey

this was the hardest problem in the section and the longest grind, but the conceptual core landed early and the fight was mostly about translating it into working code.

the breakthrough came before any code: i worked out **how much water sits on a single position**. water above position `i` is bounded by the tallest wall to its left and the tallest wall to its right, capped at the shorter of the two (the taller side can't hold water past where the shorter side leaks). so `water_at[i] = min(maxLeft[i], maxRight[i]) - height[i]`, clamped to ≥ 0, and the total is the sum over every `i`. i verified this by hand on `[0,1,0,2]` position 2: tallest left = 1, tallest right = 2, min = 1, minus the bar (0) = 1 unit.

a confusion i had to clear: i thought i'd need to detect "pools" capped by two bars and compute their area separately. the realization was that **a pool is just its columns stacked side by side** — summing per-position water automatically sums the pools, including pools spanning many bars and multiple separate pools. `maxLeft[i]` and `maxRight[i]` already encode which walls cap position `i`, so the pool boundaries fall out of the formula. no special handling needed.

i built the **O(n)-space version first** (precompute `maxLeft[]` and `maxRight[]` arrays, then sum). the bugs here were all variable-crossing and Python mechanics: `range(height)` instead of `range(len(height))`, `[] * len` making an empty list instead of `[0] * len`, the left pass updating `maxR` instead of `maxL`, seeding `maxL = height[-1]` (a right-side value), and forgetting the third summing loop entirely. once untangled it passed 22/22.

then the part i'm most happy about: i asked **"this isn't optimal on space, shouldn't we use two pointers?"** — and this time the space instinct was *correct* (unlike the 3Sum sort situation, where avoiding sort was wrong). i derived the O(1) two-pointer version. the key insight i proved myself: you process the side with the *smaller* running max, because that side's water is fully determined — if `leftMax < rightMax`, the true right max can only be ≥ rightMax > leftMax, so the min is locked to leftMax regardless of unexplored bars between the pointers. when asked whether an unknown tall bar between the pointers could change the water at `l`, i answered correctly: no, because leftMax is already the binding constraint. that's the whole proof.

---

# Attempt 1 (broken structure)

```python
class Solution:
    def trap(self, height: List[int]) -> int:
        vol = 0
        l, r = 0, 1:
        while l < r and r < len(heights):
            while heights[l] > heights[r]:
                r += 1
            while l < r:
                volHei = min(l, r)
                l += 1
                if volHei - heights[l] > 0:
                    area += vol - heights[l]
            vol = max(vol, area)
            l += 1
            r += 1
```

**what went wrong:** the structure didn't correspond to any correct algorithm. `l, r = 0, 1:` stray colon, `heights` vs the real param `height`, `area`/`vol` used before assignment, `min(l, r)` on indices. This was built before the per-position model was solid, so it got scrapped and rebuilt from the concept up.

---

# Attempt 2 (O(n) space — building the arrays, many bug rounds)

The model was now clear: build `maxLeft[]` and `maxRight[]`, then sum `min(maxLeft[i], maxRight[i]) - height[i]`. The bugs were all mechanics:

- `for i in range(height)` → `TypeError` (needs `range(len(height))`)
- `maxRight = [] * len(height)` → empty list (needs `[0] * len(height)` to be indexable)
- left pass updated `maxR` instead of `maxL` (the tracker appended and the tracker updated must match)
- `maxL = height[-1]` (seeding the left tracker with the last element, a right-side value)
- missing the final summing loop entirely → returned `None`

---

# Attempt 3 (O(n) space — Accepted)

```python
class Solution:
    def trap(self, height: List[int]) -> int:
        maxLeft = []
        maxRight = [0] * len(height)
        maxR = 0
        maxL = 0
        area = 0
        for i in range(len(height)):
            maxLeft.append(maxL)
            maxL = max(height[i], maxL)
        for i in range(len(height)-1, -1, -1):
            maxRight[i] = maxR
            maxR = max(height[i], maxR)
        for i in range(len(height)):
            cur = min(maxLeft[i], maxRight[i]) - height[i]
            if cur > 0:
                area += cur
        return area
```

**Accepted, 22/22.** Three linear passes: build `maxLeft` left-to-right, build `maxRight` right-to-left, sum the clamped per-position water. Exclusive arrays (don't include position `i`), so the `if cur > 0` clamp is required.

---

# Attempt 4 (O(1) space two-pointer — Accepted, optimal)

```python
class Solution:
    def trap(self, height: List[int]) -> int:
        res = 0
        l, r = 0, len(height)-1
        maxR, maxL = height[r], height[l]
        while l < r:
            if maxL < maxR:
                l += 1
                maxL = max(maxL, height[l])
                res += maxL - height[l]
            else:
                r -= 1
                maxR = max(maxR, height[r])
                res += maxR - height[r]
        return res
```

**Accepted, 22/22, memory beats 100%.** Two running maxes replace the two arrays. Process the smaller-max side, move pointer, update max, add water. No clamp needed because the max is updated before subtracting (so `maxL >= height[l]` always).

Bug rounds here were typos: `MaxL` vs `maxL` (Python is case-sensitive → NameError), and `(maxL, height[l])` instead of `max(maxL, height[l])` (a tuple, not a max call).

---

# Accepted Solution(s)

**Optimal (O(1) space):**

```python
class Solution:
    def trap(self, height: List[int]) -> int:
        res = 0
        l, r = 0, len(height)-1
        maxR, maxL = height[r], height[l]
        while l < r:
            if maxL < maxR:                      # left side is the binding constraint
                l += 1
                maxL = max(maxL, height[l])      # update FIRST → no negative water
                res += maxL - height[l]
            else:                                # right side is binding
                r -= 1
                maxR = max(maxR, height[r])
                res += maxR - height[r]
        return res
```

---

# Why It Works

**Per-position model:** water on top of position `i` is bounded by the tallest wall to its left and the tallest to its right, capped at the *shorter* of the two (water spills over the lower wall). So `water_at[i] = min(maxLeft[i], maxRight[i]) - height[i]`, clamped to ≥ 0. The total is the sum over all `i`.

**Pools dissolve into columns:** a pool capped by two bars is just adjacent columns of water stacked side by side. Summing per-position water automatically sums whole pools — including multi-bar pools and multiple separate pools — because `maxLeft[i]`/`maxRight[i]` encode which walls confine each position. No explicit pool detection.

**The two-pointer invariant:** process the side with the *smaller* running max. If `maxL < maxR`, the water at `l` is exactly `maxL - height[l]`, because the true right max can only be ≥ `maxR > maxL`, so `min(maxL, trueRightMax) = maxL` no matter what unexplored bars sit between the pointers. The shorter side is always safe to commit; the taller side can only get taller, so it never becomes the binding constraint for the position being resolved.

---

# Traversal Logic

**O(n) version:** three independent linear passes — left-to-right (build `maxLeft`), right-to-left (build `maxRight`), left-to-right (sum). No convergence.

**O(1) version:** two pointers converging, processing the smaller-max side each step.

```
height = [0, 1, 0, 2]
          l        r        maxL=0, maxR=2 → maxL<maxR, process left
          → l moves right, maxL updates, add water at new l
... pointers converge, always resolving the side with the smaller running max ...
```

This is a distinct two-pointer flavor from the rest of the section:
- **125:** unconditional convergence
- **167 / 15:** sum-driven movement
- **11:** min-of-two-walls greedy
- **42:** smaller-running-max-driven (resolve the side you have enough info to commit)

---

# Business Logic

The decisions made at each step — separate from how we move.

- compute per-position water via the shorter-wall cap
- clamp negatives (O(n) version) or avoid them by updating max first (O(1) version)
- accumulate into the running total
- (O(1) version) decide which side to resolve: the one with the smaller running max

---

# Variable Roles

**O(1) version:**

| Variable | Represents | Stores | How It Changes | Why It Exists |
|----------|------------|--------|----------------|---------------|
| `l` | left pointer | int index | `+= 1` when left is binding | left scan position |
| `r` | right pointer | int index | `-= 1` when right is binding | right scan position |
| `maxL` | tallest bar seen from the left so far | int | `max(maxL, height[l])` | left wall cap for the current position |
| `maxR` | tallest bar seen from the right so far | int | `max(maxR, height[r])` | right wall cap |
| `res` | total trapped water | int | `+= per-position water` | the answer |

---

# Visual Walkthrough

Input: `height = [0, 1, 0, 2]` (O(1) two-pointer version)

| Step | l | r | maxL | maxR | which side | water added | res |
|------|---|---|------|------|-----------|-------------|-----|
| seed | 0 | 3 | 0 | 2 | — | — | 0 |
| 1 | 0→1 | 3 | max(0,1)=1 | 2 | maxL<maxR → left | 1-1=0 | 0 |
| 2 | 1→2 | 3 | max(1,0)=1 | 2 | maxL<maxR → left | 1-0=1 | 1 |
| 3 | 2→3? | — | — | — | l==r, stop | — | 1 |

Answer: **1**.

Why the shorter wall caps the water:

```
        █          maxRight
    █ ~ █          water at the dip = min(left wall, right wall) - bar
    █ █ █
   index            the '~' fills only to the shorter confining wall
```

---

# Optimization Journey

1. **Brute Force:** for each position, scan all the way left for the max and all the way right for the max, apply the formula. O(n²) time, O(1) space.
2. **Bottleneck:** recomputing `maxLeft`/`maxRight` from scratch at every position, even though they barely change as you slide.
3. **Observation A (precompute):** build `maxLeft[]` in one left pass and `maxRight[]` in one right pass, then look both up in O(1). → **O(n) time, O(n) space.**
4. **Observation B (collapse arrays):** you only ever use the two maxes at the *current* position. With two pointers from both ends and two running maxes, you can resolve the side with the smaller max using only scalars — the arrays vanish. → **O(n) time, O(1) space.**
5. **Final Approach:** the two-pointer version. The interviewer-facing story is the full ladder: brute force → precompute arrays → two-pointer O(1).

---

# Pattern Recognition

## Signal Phrases

- "trapped water" / "elevation map" → per-position water bounded by left/right maxes
- middle bars matter (vs Container) → sum over all positions, not one pool
- O(1) space target → two-pointer collapse of the precompute arrays

## Pattern Used

Two Pointers (smaller-running-max-driven) for O(1); precompute arrays for the O(n) intermediate.

## Why This Pattern Fits

- per-position water is a local formula needing only two maxes
- those maxes can be tracked with two converging pointers
- the smaller-max side is always safe to resolve → single pass, O(1) space

---

# Pattern Comparison

| Pattern | Why It Fits | Why It Doesn't | Verdict |
|---------|-------------|----------------|---------|
| Two Pointers (O(1)) | resolves smaller-max side with scalars | — | ✅ optimal |
| Precompute Arrays (O(n) space) | direct write-out of the formula | O(n) space | ⚠️ correct, not optimal |
| Brute Force | trivially correct | O(n²) | ❌ too slow |
| Monotonic Stack | also O(n), pops on rising bars | more complex, computes water in horizontal slabs | ⚠️ valid alternative, trickier |

---

# Common Mistakes

The ones i actually hit:

- building before the per-position model was solid (Attempt 1 had no valid structure)
- `range(height)` instead of `range(len(height))`
- `[] * len` (empty) instead of `[0] * len` (indexable)
- crossing trackers: left pass updating `maxR` instead of `maxL`
- seeding `maxL = height[-1]` (a right-side value for the left tracker)
- forgetting the final summing loop / `return`
- case sensitivity: `MaxL` vs `maxL` → NameError
- `(a, b)` instead of `max(a, b)` → a tuple, not a max
- (conceptual) thinking pools need separate handling — they don't, columns sum to pools

---

# Edge Cases

| Case | Input | Expected | How Handled |
|------|-------|----------|-------------|
| empty | `[]` | 0 | NeetCode guards with `if not height: return 0`; mine relies on tests not including it |
| no dips | `[1,2,3]` | 0 | each position's water is 0 |
| single dip | `[0,1,0,2]` | 1 | the middle 0 traps 1 |
| flat | `[2,2,2]` | 0 | no trapping |
| big basin | `[3,0,0,0,3]` | 9 | three columns of depth 3 sum to 9 |

---

# Debugging Lessons

- **per-position thinking beats pool-detection.** Summing columns automatically sums pools; never compute pool areas explicitly.
- **tracker/array pairing:** the variable you append/store and the variable you update must be the same family. Left pass touches only `maxL`/`maxLeft`; right pass only `maxR`/`maxRight`.
- **update-before-subtract removes the clamp:** updating the running max to include the current bar before computing water guarantees non-negative water — the O(1) version needs no `if > 0`.
- **Python mechanics, again:** case sensitivity (`MaxL` ≠ `maxL`), `max(a,b)` needs the keyword (`(a,b)` is a tuple), `[0]*n` vs `[]*n`.
- **the space instinct was right this time:** asking "can we drop to O(1)?" led to the optimal solution. Contrast with 3Sum, where avoiding the sort to "save space" was wrong. The skill is telling the two situations apart.

---

# NeetCode / Official Solution

## Code (O(1) two-pointer)

```python
class Solution:
    def trap(self, height: List[int]) -> int:
        if not height: return 0
        l, r = 0, len(height) - 1
        leftMax, rightMax = height[l], height[r]
        res = 0
        while l < r:
            if leftMax < rightMax:
                l += 1
                leftMax = max(leftMax, height[l])
                res += leftMax - height[l]
            else:
                r -= 1
                rightMax = max(rightMax, height[r])
                res += rightMax - height[r]
        return res
```

## Comparison to My Solution

| | NeetCode | Mine (O(1)) |
|---|----------|-------------|
| Algorithm | two pointers, smaller-max-driven | identical |
| Variable names | `leftMax` / `rightMax` | `maxL` / `maxR` |
| Empty guard | `if not height: return 0` | none (passed tests anyway) |
| Move/update/add order | move → update → add | same |
| Time / Space | O(n) / O(1) | O(n) / O(1) |

Identical algorithm. The only meaningful difference is NeetCode's empty-array guard (`if not height: return 0`), which prevents a crash on `[]` (seeding `height[l]` on an empty list throws). Mine omits it and passed because the test suite has no empty input — but the guard is the more defensive habit worth adopting.

---

# Deep Understanding

The non-obvious core is **why processing the smaller-max side is always safe**. When `maxL < maxR` at position `l`, the water there is `min(maxL, trueRightMax) - height[l]`. You don't know `trueRightMax` (there may be taller bars between the pointers), but you don't need to: it's at least `maxR`, and `maxR > maxL`, so `min(maxL, trueRightMax) = maxL` unconditionally. The left running max alone determines the water. The taller side carries no risk of becoming the binding constraint because it can only grow. This is what lets you discard the two precompute arrays — you never need the full right-max picture for a left-resolved position, just the knowledge that the right side is "at least as tall," which the running max already guarantees.

The second subtlety is **update-before-subtract**: by setting `maxL = max(maxL, height[l])` before `res += maxL - height[l]`, the running max is inclusive of the current bar, so the subtraction is never negative and no clamp is needed.

---

# Complexity Analysis

| Version | Time | Space | Reason |
|---------|------|-------|--------|
| Brute force | O(n²) | O(1) | re-scan both directions per position |
| Precompute arrays | O(n) | O(n) | three passes, two stored arrays |
| Two pointers | O(n) | O(1) | single pass, two scalar maxes |

---

# Related Problems

| # | Problem | Difficulty | Same Pattern? | Notes |
|---|---------|------------|---------------|-------|
| 11 | Container With Most Water | Medium | Related two pointers | one max pool, middle bars ignored; min-of-walls greedy |
| 84 | Largest Rectangle in Histogram | Hard | Monotonic stack | sibling of the stack approach to 42 |
| 407 | Trapping Rain Water II | Hard | Heap + BFS | 2D version, much harder |
| 238 | Product of Array Except Self | Medium | Prefix/suffix arrays | same precompute-both-directions idea |

---

# Interview Explanation (30 Seconds)

Water on each position is capped by the shorter of the tallest wall to its left and right, minus the bar itself. I use two pointers from both ends with two running maxes, always processing the side whose max is smaller — because that side's water is fully determined regardless of what's between the pointers. Single pass, O(n) time, O(1) space.

---

# Interview Explanation (2 Minutes)

Each position traps `min(maxLeft, maxRight) - height[i]`, clamped at zero, because water can't rise above the shorter of the two confining walls. Total is the sum over all positions; I don't detect pools because summing columns sums pools automatically.

The naive way precomputes a left-max array and a right-max array in two passes, then sums — O(n) time but O(n) space. To get O(1) space I use two pointers from the ends with two running maxes. The key is that I process whichever side has the *smaller* running max. If `leftMax < rightMax`, the water at the left pointer is exactly `leftMax - height[l]`, because the true right max is at least `rightMax`, which already exceeds `leftMax`, so the min is pinned to `leftMax` no matter what bars I haven't seen between the pointers. The shorter side is always safe to commit; the taller side can only grow taller, so it never becomes the binding constraint.

I update the running max before subtracting, which keeps the water non-negative without a clamp. It's O(n) time, single pass, O(1) space. The main alternative is a monotonic stack that fills water in horizontal slabs — also O(n) but trickier to reason about.

---

# Common Follow-Up Questions

- **"Can you do it in O(1) space?"** → yes, the two-pointer version above; this is the whole reason it sits in the two-pointers section.
- **"Why is processing the smaller-max side correct?"** → the binding wall is the shorter side; the taller side can only grow, so the min for the resolved position is fixed.
- **"Why don't you need to handle pools separately?"** → a pool is adjacent columns stacked; per-position sums equal pool sums.
- **"What's the monotonic-stack approach?"** → push indices of decreasing bars; when a taller bar arrives, pop and add the horizontal water slab between the boundaries. Also O(n).
- **"Extend to 2D (LC 407)?"** → min-heap of the boundary cells, BFS inward from the lowest wall.

---

# Key Takeaways

- water per position = `min(maxLeft, maxRight) - height[i]`, clamped ≥ 0; sum over all positions.
- pools dissolve into columns — never detect pools explicitly.
- the array-to-scalar collapse: track two running maxes with two pointers, resolve the smaller-max side → O(1) space.
- update the running max before subtracting to avoid negative water.
- the "can we go O(1)?" instinct was right here — and knowing when it's right (vs the 3Sum sort) is the meta-skill.

---

# Revision Notes

When re-reading in a week/month, the minimum to refresh:
- formula: per-position water = shorter confining wall minus the bar; sum all.
- O(n) version: two precompute arrays + a summing loop (exclusive → needs clamp).
- O(1) version: two pointers, two running maxes, resolve the smaller-max side, update-before-subtract (no clamp).
- the one proof to recall: smaller-max side is safe because the taller side can only grow and never becomes the binding constraint.

---

# Final Mental Model

Imagine walking inward from both ends of the skyline, carrying a memory of the tallest wall you've passed on each side. At every step you stand on whichever side has the *shorter* tallest-wall-so-far, because on that side you already know exactly how deep the water is — the other side is guaranteed at least as tall, so it can't lower the water line. You pour in `(your tallest wall) - (current ground)` and step inward. You never need a map of the whole skyline, just the two tallest walls behind you — which is how the whole problem collapses from two big arrays down to two numbers.
