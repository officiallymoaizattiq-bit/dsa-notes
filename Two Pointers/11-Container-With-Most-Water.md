# 11. Container With Most Water

## Problem

Given an integer array `heights` where `heights[i]` is the height of a vertical bar at position `i`, pick **two** bars to form a container and return the maximum water it can hold. The water is a 2D cross-section: its volume is the distance between the two bars times the height of the **shorter** bar.

Concrete example:

```
heights = [1, 8, 6, 2, 5, 4, 8, 3, 7]

Best container: bars at index 1 (height 8) and index 8 (height 7)
width  = 8 - 1 = 7
height = min(8, 7) = 7
area   = 7 * 7 = 49
```

---

# My Learning Journey

before touching code i worked through the *mental model* of what a container even is, because the problem wording is easy to misread. the key thing that had to click: **the water level is capped by the shorter of the two walls.** if one wall is 8 and the other is 3, water spills over the 3 side — the tall wall's extra height holds nothing. so area is `width × min(left, right)`, and the bars *between* the two chosen walls are irrelevant (that's a different problem, trapping rain water).

once the model was solid, i went straight for two pointers (correct instinct). my first attempt had the right skeleton but several bugs, and one revealed the model had slipped slightly — i wrote `min(l, r)` using the *indices* instead of `min(heights[l], heights[r])` using the *heights*. i also repeated the `if`-vs-`elif` tangle from earlier problems (two independent `if`s letting both pointers move), computed the area after moving instead of before, and collected every area into a list to sort at the end instead of keeping a running max.

after fixing all four in one pass, the solution was accepted. the part i had to actually articulate — and the reason this problem is two pointers and not brute force — is **why you move the shorter wall.** i reasoned it out in my own words: moving the taller wall shrinks the width while staying capped by the same short wall, so it's guaranteed worse. moving the shorter wall also shrinks the width, but it's the *only* move that could land on a taller wall and increase the height. the precise version: moving the shorter wall isn't guaranteed better, it's the only move with any *upside* — the tall-wall move has zero upside, so you keep the move that preserves hope.

---

# Attempt 1

```python
class Solution:
    def maxArea(self, heights: List[int]) -> int:
        vol = []
        l, r = 0, len(heights)-1
        while l < r:
            if heights[l] > heights[r]:
                r -= 1
            if heights[r] > heights[l]:
                l += 1
            vol.append((r-l) * min(l, r))
        vol.sort()
        return vol[-1]
```

**what went wrong (several bugs):**
- **`min(l, r)` uses indices, not heights.** Water height is the shorter *bar* → must be `min(heights[l], heights[r])`. This was the model slipping.
- **two independent `if`s** let both pointers move in one iteration, and the second reads the already-moved pointer. Should be one `if/else`.
- **move direction tangled** — the logic should be "move the shorter wall," cleanly.
- **area computed after moving** — should compute the current pair first, then move.
- **`vol` list + sort** is O(n log n) time and O(n) space for nothing — a running max is O(1).

---

# Attempt 2 (Accepted)

```python
class Solution:
    def maxArea(self, heights: List[int]) -> int:
        res = 0
        l, r = 0, len(heights)-1
        while l < r:
            res = max((r-l) * min(heights[l], heights[r]), res)
            if heights[r] < heights[l]:
                r -= 1
            else:
                l += 1
        return res
```

**what i fixed (all four at once):**
- height term is `min(heights[l], heights[r])` (heights, not indices)
- single `if/else`, moving the shorter wall
- area computed *before* moving
- running max instead of a list + sort

Accepted. The move logic is mirrored from the skeleton (checks "right shorter → move right") but equivalent: it always moves the shorter wall, and the equal case moving `l` is harmless.

---

# Accepted Solution(s)

```python
class Solution:
    def maxArea(self, heights: List[int]) -> int:
        res = 0
        l, r = 0, len(heights) - 1
        while l < r:
            res = max((r - l) * min(heights[l], heights[r]), res)  # width × shorter wall
            if heights[r] < heights[l]:   # right is the shorter wall
                r -= 1                     # drop it — it's maxed out
            else:                          # left is shorter (or equal)
                l += 1
        return res
```

---

# Why It Works

**The greedy pruning argument.** At any `(l, r)`, the area is capped by the shorter wall. Suppose `heights[l]` is shorter. Every future pair still involving `l` is *narrower* (since `r` only moves left), and its height is still capped at *at most* `heights[l]` (the short wall bounds it no matter the partner). So every remaining container using `l` is `≤` the one you just computed — `l` is exhausted, it can never beat its current showing. You discard it by moving `l` inward, losing nothing provably.

Moving the *taller* wall instead would be the mistake: narrower width, same short-wall cap, strictly worse, and you'd never test whether a taller partner for the short wall exists.

**The invariant:** `res` always holds the best area seen so far, and every discarded pair was provably eliminated. When the pointers meet, every container that could win has been considered.

---

# Traversal Logic

How we move through the data — two pointers converging, direction driven by the heights.

```
heights = [1, 8, 6, 2, 5, 4, 8, 3, 7]
           l                          r      width=8, min(1,7)=1, area=8
           l is shorter → move l

heights = [1, 8, 6, 2, 5, 4, 8, 3, 7]
              l                       r      width=7, min(8,7)=7, area=49 ← best
                                             r is shorter → move r
... continue converging, each step dropping the shorter wall ...
```

This is the third flavor of two-pointer movement seen in this section:
- **125 Valid Palindrome:** unconditional (both always step inward)
- **167 / 15:** sum-driven (the sum vs target decides)
- **11 Container:** min-driven (the shorter wall decides)

---

# Business Logic

The decisions made at each step — separate from how we move.

- compute `width × min(heights[l], heights[r])`
- update the running max
- decide which wall to drop: always the **shorter** one (it's the bottleneck and it's maxed out)

The area computation and the running max are business. The "drop the shorter wall" move is traversal. The greedy justification links them.

---

# Variable Roles

| Variable | Represents | Stores | How It Changes | Why It Exists |
|----------|------------|--------|----------------|---------------|
| `l` | left wall pointer | int index | `+= 1` when left is shorter | left boundary of the container |
| `r` | right wall pointer | int index | `-= 1` when right is shorter | right boundary of the container |
| `res` | best area so far | int | `max(res, area)` each step | running maximum, avoids storing all areas |

---

# Visual Walkthrough

Input: `heights = [1, 8, 6, 2, 5, 4, 8, 3, 7]`

| Step | l | r | h[l] | h[r] | width | min | area | res | Move |
|------|---|---|------|------|-------|-----|------|-----|------|
| 1 | 0 | 8 | 1 | 7 | 8 | 1 | 8 | 8 | l shorter → l++ |
| 2 | 1 | 8 | 8 | 7 | 7 | 7 | **49** | 49 | r shorter → r-- |
| 3 | 1 | 7 | 8 | 3 | 6 | 3 | 18 | 49 | r shorter → r-- |
| 4 | 1 | 6 | 8 | 8 | 5 | 8 | 40 | 49 | equal → r-- |
| 5 | 1 | 5 | 8 | 4 | 4 | 4 | 16 | 49 | r-- |
| ... | | | | | | | | 49 | converges, res stays 49 |

Answer: **49**.

Why the short wall caps the height:

```
8 |█
  |█
3 |█  ~  ~  █   ← right wall = 3, water fills only to 3
  |█  ~  ~  █     left wall above 3 is dry — holds nothing
  +--+--+--+
   l        r    area = width × 3 (the shorter wall)
```

---

# Optimization Journey

1. **Brute Force:** check all O(n²) pairs of bars, compute each area, track the max. O(n²) time, O(1) space.
2. **Bottleneck:** the O(n²) — re-checking pairs that can't possibly beat the best.
3. **Observation:** the shorter wall is the bottleneck. Once you've measured a pair, the shorter wall can never do better with any *narrower* partner — so it can be discarded.
4. **Optimization:** start at the widest pair, repeatedly drop the shorter wall, keeping a running max.
5. **Final Approach:** O(n) time, O(1) space. An interviewer discovers it by asking "after measuring a pair, which wall is safe to abandon?"

---

# Pattern Recognition

## Signal Phrases

- "two bars to form a container" → pick a pair → two pointers
- "maximum water" → maximize an area over pairs
- area capped by the shorter side → min-driven movement
- the O(n²)-pairs space begs for a pruning trick

## Pattern Used

Two Pointers (converging, greedy, min-driven movement).

## Why This Pattern Fits

- start wide, the width only shrinks, so you want to capture wide areas early
- the shorter wall is a provable bottleneck → safe to discard → O(n)
- O(1) space with a running max

---

# Pattern Comparison

| Pattern | Why It Fits | Why It Doesn't | Verdict |
|---------|-------------|----------------|---------|
| Two Pointers (greedy) | shorter-wall pruning gives O(n) | — | ✅ optimal |
| Brute Force | trivially correct | O(n²) | ❌ too slow |
| Stack / DP | overkill here; that's for trapping rain water (LC 42) | wrong problem shape | ❌ misfit |

---

# Common Mistakes

The ones i actually hit:

- `min(l, r)` using indices instead of `min(heights[l], heights[r])`
- two independent `if`s allowing both pointers to move in one iteration
- computing the area after moving instead of before
- storing all areas in a list and sorting instead of a running max
- (conceptual trap) confusing this with trapping rain water — here the middle bars are ignored

---

# Edge Cases

| Case | Input | Expected | How Handled |
|------|-------|----------|-------------|
| two bars only | `[1, 1]` | 1 | single pair: width 1 × min(1,1) | 
| ascending | `[1, 2, 3, 4]` | 4 | wide pair (1,4) gives 3×1=3; (2,4)... best is 2×2=4 from idx1,3 — pointers find it |
| all equal | `[5, 5, 5]` | 10 | widest pair wins: 2 × 5 |
| tall narrow vs wide short | `[2, 1, 1, 1, 2]` | 8 | ends win: width 4 × min(2,2)=8 |

---

# Debugging Lessons

The model-level lesson: **the height term is the shorter bar's height, not anything about indices.** Mixing up `min(l, r)` and `min(heights[l], heights[r])` is the tell that "what does the formula actually represent" wasn't fully locked.

The structural lesson, again: **one `if/else`, not two `if`s**, whenever exactly one of two pointer moves should happen per iteration — same lesson as 167 and 15.

The efficiency lesson: **a running max replaces collect-then-sort** — keep O(1) space instead of O(n) when you only need the best value, not all values.

---

# NeetCode / Official Solution

## Code

```python
# LINEAR TIME SOLUTION: O(n)
res = 0
l, r = 0, len(height) - 1
while l < r:
    area = (r - l) * min(height[l], height[r])
    res = max(res, area)
    if height[l] < height[r]:
        l += 1
    else:
        r -= 1
return res
```

## Comparison to My Solution

| | NeetCode | Mine |
|---|----------|------|
| Algorithm | two pointers, greedy, min-driven | identical |
| Move check | `height[l] < height[r]` → move l | `heights[r] < heights[l]` → move r |
| Tie goes to | `r` (the `else`) | `l` (the `else`) |
| Area | computed into `area` var first | inline in `max(...)` |
| Time / Space | O(n) / O(1) | O(n) / O(1) |

Identical algorithm, mirror-image phrasing of the move condition. NeetCode checks "left shorter → move left," mine checks "right shorter → move right" — both always drop the shorter wall. The tie case differs (NeetCode moves `r`, mine moves `l`) but is irrelevant: equal walls give the same cap either direction. No difference in output or complexity.

---

# Deep Understanding

The non-obvious part is the precise form of the greedy argument: moving the shorter wall is **not** guaranteed to find a bigger area. You might move `l` from height 1 to height 0 and do worse. The justification is about *upside*, not guarantee:

- moving the **taller** wall: width shrinks, height still capped by the unchanged short wall → **zero** upside, strictly worse-or-equal.
- moving the **shorter** wall: width shrinks, but the height cap *could* rise if the new wall is taller → **possible** upside.

You keep the move that preserves the possibility of improvement and discard the one that's already maxed out. That's why it's a valid greedy choice — you only ever abandon a wall you've proven can't beat its current best with any narrower partner.

---

# Complexity Analysis

| Metric | Complexity | Reason |
|--------|------------|--------|
| Time | O(n) | `l` and `r` only move toward each other, never reset; combined ≤ n steps |
| Space | O(1) | one `res` variable and two pointers — the payoff of dropping the list+sort |

---

# Related Problems

| # | Problem | Difficulty | Same Pattern? | Notes |
|---|---------|------------|---------------|-------|
| 42 | Trapping Rain Water | Hard | Related two pointers | the hard sibling; middle bars DO count, trickier movement |
| 167 | Two Sum II | Medium | Yes (two pointers) | sum-driven movement instead of min-driven |
| 15 | 3Sum | Medium | Yes | two-pointer scan inside an outer loop |
| 75 | Sort Colors | Medium | Yes | different two-pointer flavor (partitioning) |

---

# Interview Explanation (30 Seconds)

I use two pointers at the widest pair. The area is width times the shorter wall, so I compute it, keep a running max, then move whichever pointer is at the shorter wall. The shorter wall is the bottleneck — it can't do better with any narrower partner — so dropping it is safe, giving O(n) time, O(1) space.

---

# Interview Explanation (2 Minutes)

The area between two bars is the distance between them times the shorter bar's height, because water spills over the lower wall. So I want the pair maximizing `width × min(heights)`. There are O(n²) pairs, but I can avoid checking them all.

I start with the widest possible pair — pointers at both ends. I compute its area and record it. Then I ask which wall to discard. The shorter wall is the bottleneck: any future pair that still uses it is narrower (width only shrinks) and still capped at the short wall's height, so it can never beat what I just measured. I discard the shorter wall by moving its pointer inward. The taller wall I keep, because moving it would shrink the width while staying capped by the same short wall — strictly worse.

I repeat until the pointers meet, keeping a running max. It's O(n) because the pointers only move inward and never reset, and O(1) space for the single max variable. The subtle point worth stating is that moving the shorter wall isn't *guaranteed* to improve the area — it's just the only move with any upside, since the taller-wall move has none.

---

# Common Follow-Up Questions

- **"Why move the shorter wall?"** → the #1 follow-up. The shorter wall is the bottleneck; it can't beat its current area with any narrower partner, so it's safe to discard. The taller-wall move has zero upside.
- **"Is this greedy? Prove it's correct."** → yes; each discard provably eliminates only pairs that can't beat the current best.
- **"Why O(n) and not O(n²)?"** → pointers are monotonic, never reset; total movement ≤ n.
- **"How is this different from Trapping Rain Water?"** → here the middle bars are ignored and you want one max container; there every bar traps water above it and you sum all of it.

---

# Key Takeaways

- area = `width × min(heights[l], heights[r])` — the shorter wall caps the water.
- move the **shorter** wall: it's the only move with upside (taller-wall move is strictly worse).
- this is greedy two pointers — each discard is provably safe.
- keep a running max, not a list to sort — O(1) space.

---

# Revision Notes

When re-reading in a week/month, the minimum to refresh:
- skeleton: pointers at both ends → `area = (r-l) * min(h[l], h[r])` → running max → move the shorter wall → stop when they meet.
- the one insight: shorter wall is the bottleneck, discarding it is safe because no narrower partner can beat its current area.
- the precise framing: move the shorter wall because it's the only move with *upside*, not because it's guaranteed better.

---

# Final Mental Model

Two people hold up walls at opposite ends of a tank and you pour water between them — it fills only as high as the shorter wall. To find the most water, start with the walls as far apart as possible, then repeatedly send the *shorter* wall inward looking for a taller one, because the tall wall is already doing all it can and only the short wall holds you back. The widest start plus always-retire-the-bottleneck is what lets you find the best container in a single sweep instead of trying every pair.
