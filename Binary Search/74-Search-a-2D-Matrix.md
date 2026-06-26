# 74. Search a 2D Matrix

## Problem
Given an `m x n` matrix where each row is sorted left-to-right **and** the first value of every row is greater than the last value of the previous row, return whether `target` exists in the matrix. Expected `O(log(m * n))`.

Example: `matrix = [[1,3,5,7],[10,11,16,20],[23,30,34,60]]`, `target = 3` → `True`. `target = 13` → `False`.

## Learning Journey
No worked solve happened in this chat — the solution was pasted directly and notes were requested. So there's no hint-ladder progression, no attempt history, and no debugging arc to record. These notes are built from the pasted solution and the explanation that followed. The one idea worth anchoring: because the rows chain (last of each row < first of the next), the whole matrix is a sorted 1D array folded into a grid, which is *why* binary search applies twice.

## Accepted Solution
```python
class Solution:
    def searchMatrix(self, matrix: List[List[int]], target: int) -> bool:
        ROWS, COLS = len(matrix), len(matrix[0])

        top, bot = 0, ROWS - 1
        while top <= bot:
            row = (top + bot) // 2
            if target > matrix[row][-1]:
                top = row + 1
            elif target < matrix[row][0]:
                bot = row - 1
            else:
                break

        if not (top <= bot):
            return False
        row = (top + bot) // 2
        l, r = 0, COLS - 1
        while l <= r:
            m = (l + r) // 2
            if target > matrix[row][m]:
                l = m + 1
            elif target < matrix[row][m]:
                r = m - 1
            else:
                return True
        return False
```
Result: Accepted. (Runtime not recorded this session — and LeetCode's runtime percentile is noisy anyway, so don't read into it.)

## Why It Works
**Invariant:** if `target` exists, it lives in exactly one row — the row whose range `[matrix[row][0], matrix[row][-1]]` contains it — so once you've isolated that row, the rest of the matrix is irrelevant.

**Key operation:** comparing `target` against a row's *endpoints* lets you throw away half the rows each step (above range → go down, below range → go up).

**Load-bearing line:** `if not (top <= bot): return False`. The row loop only `break`s when it lands inside a row's range. If it never does, it exits the `while top <= bot` loop the normal way, which means `top > bot` is now true — and that's exactly the "no row could contain target" case. Without this guard you'd run pass 2 on a garbage row.

## How It Works
**Traversal:** two independent binary searches.
```
Pass 1 (rows):     top --------> row <-------- bot     (collapses to the candidate row)
Pass 2 (one row):  l ----------> m <---------- r       (standard binary search in that row)
```

**Business (decision each step):**
1. Pass 1: is `target` past this row's last element? → search lower rows. Before its first element? → search higher rows. Otherwise it's in this row's range → stop.
2. Guard: did pass 1 ever land inside a row? If not → `False`.
3. Pass 2: ordinary binary search across the candidate row's columns. Match → `True`, exhaust → `False`.

**Variables:**
| Variable | Represents | Stores | How It Changes | Why It Exists |
|----------|------------|--------|----------------|---------------|
| `ROWS`, `COLS` | matrix dimensions | ints | fixed | bounds for both searches |
| `top`, `bot` | row search window | row indices | `top=row+1` or `bot=row-1` | narrows to the candidate row |
| `row` | current/candidate row | row index | recomputed as midpoint | the one row target could be in |
| `l`, `r` | column search window | col indices | `l=m+1` or `r=m-1` | narrows to the target column |
| `m` | column midpoint | col index | `(l+r)//2` each pass | element under test in pass 2 |

## Visual Walkthrough
Input: `matrix = [[1,3,5,7],[10,11,16,20],[23,30,34,60]]`, `target = 3`.

**Pass 1 — find the row** (`top=0, bot=2`):
| Step | top,bot | row | Row Range | Check | Action |
|------|---------|-----|-----------|-------|--------|
| 1 | 0,2 | 1 | [10..20] | 3 < 10 | `bot = 0` |
| 2 | 0,0 | 0 | [1..7] | 1 ≤ 3 ≤ 7 | `break` |

Guard: `top(0) <= bot(0)` → don't return False. Candidate `row = 0`.

**Pass 2 — search row `[1,3,5,7]`** (`l=0, r=3`):
| Step | l,r | m | matrix[0][m] | Check | Action |
|------|-----|---|--------------|-------|--------|
| 1 | 0,3 | 1 | 3 | 3 == 3 | `return True` |

## Optimization Journey
1. **Brute force:** scan every cell → `O(m * n)`.
2. **Bottleneck:** ignores that the data is sorted; re-checks values that can't possibly match.
3. **Observation:** rows chain, so the matrix is one globally sorted sequence — both axes are searchable by halving.
4. **Optimization:** binary search the rows by their endpoints, then binary search inside the surviving row.
5. **Final:** an interviewer gets here by asking "the data's fully sorted — can I discard half of it each step?" Yes, on both axes → `O(log m + log n)`.

## Pattern Recognition
**Signals:** "sorted matrix" + "find/search target" + expected better-than-linear → Binary Search. "first int of each row > last int of previous row" → globally sorted → 1D binary search logic applies to a 2D grid.

**Pattern used:** Binary Search — the sorted-and-chained structure gives a discardable half at every step on both axes.

| Pattern | Why It Fits | Why It Doesn't | Verdict |
|---------|-------------|----------------|---------|
| Binary Search | sorted on both axes, log target complexity | — | optimal |
| Linear scan | trivially correct | `O(m*n)`, misses the target complexity | too slow |
| Staircase two-pointer (top-right) | works when rows *don't* chain (LC 240) | here it's `O(m+n)`, worse than binary search | suboptimal for this variant |

## Edge Cases & Debugging
Clean paste — no bugs surfaced, nothing for the recurring-bug tracker. Edge behavior the code already handles correctly:
| Case | Input | Expected | How Handled |
|------|-------|----------|-------------|
| Target below everything | target < `matrix[0][0]` | False | row loop drives `bot` past `top`, guard returns False |
| Target above everything | target > `matrix[-1][-1]` | False | row loop drives `top` past `bot`, guard returns False |
| Target between two rows' ranges | e.g. `13` | False | no row's range contains it → guard returns False |
| Single row / single col | `[[5]]` | depends | both loops still run correctly |

Note: assumes `matrix[0]` exists. LeetCode 74's constraints guarantee `m, n ≥ 1`, so no empty-matrix guard is needed — but flag that assumption out loud in an interview.

## Complexity
| Metric | Complexity | Reason |
|--------|------------|--------|
| Time | `O(log m + log n)` = `O(log(m*n))` | one binary search over rows, one over columns |
| Space | `O(1)` | only index variables |

## Interview Prep
**30s:** Since each row's first element exceeds the previous row's last, the matrix is a sorted array folded into a grid. Binary search the rows by their endpoints to isolate the one possible row, then binary search inside it. `O(log(m*n))` time, `O(1)` space.

**2 min:** Recognition signal — fully sorted matrix plus a "find target" ask with sub-linear expectations screams binary search. Invariant: target, if present, lives in exactly one row whose `[first, last]` range covers it. Key operation: comparing target to a row's endpoints discards half the rows per step; then a standard binary search inside the candidate row. Complexity `O(log m + log n)`. Tradeoff: the two-pass version is the most intuitive, but you can collapse it into a single binary search over `m*n` by mapping flat index `i → (i // COLS, i % COLS)` — less code, same complexity.

**Follow-ups:**
- *What if rows don't chain (only each row and each column is sorted)?* → That's LC 240; binary search no longer applies globally. Walk from the top-right corner, moving left or down, `O(m+n)`.
- *Return the insertion position instead of a bool?* → return `l` after the inner loop exits.

## Revision
**1 week:** Re-state the invariant (target lives in exactly one row whose first/last bracket it) and re-trace a *not-found* input — `target = 13` on the example matrix — to confirm the guard fires.

**1 month:** Skeleton — binary search rows by endpoints → guard → binary search the row. Complexity `O(log(m*n))` because you halve rows then halve columns.

**Mental model:** It's a phone book folded into a grid. First flip to the right *page* by glancing at the first and last name on each page (the row endpoints), then binary-search names *within* that page. Two flips of the same trick.

**Takeaways:** When a 2D structure is globally sorted via row-chaining, treat it as 1D. Binary search needs two things — a sorted axis and a rule to discard half each step; comparing against row *endpoints* (not individual cells) is what makes the row pass `O(log m)` instead of `O(m)`.
