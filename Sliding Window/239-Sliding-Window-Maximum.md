# 239. Sliding Window Maximum

## Problem
Given an array `nums` and a window size `k`, the window slides left to right one position at a time. For each window position, return the maximum value inside it.

Example: `nums = [1,3,-1,-3,5,3,6,7]`, `k = 3` → `[3,3,5,5,6,7]`
The six windows are `[1,3,-1] [3,-1,-3] [-1,-3,5] [-3,5,3] [5,3,6] [3,6,7]`, maxes `3 3 5 5 6 7`.

## Learning Journey
Started with the brute force: march a fixed-size window and rescan it for the max each step. First paste had two problems — the loop body was dedented (paste mangling) and, more importantly, the bound was `while r < len(nums)`, which **systematically dropped the last window** every time. Traced the breaking input, saw the missing `[3,6,7] → 7`, fixed the bound to `<=`. Brute force then correct but O(n·k) — it rescans `k` elements per window and throws away everything it learned about the previous one.

For the optimization, worked the core observation by walking the array and asking "can this element ever be the max of a *future* window?" — once a bigger element appears to its right, the smaller one is permanently dominated (any later window holding the small one also holds the big one). That gives a shrinking, strictly-decreasing list of still-relevant candidates.

Translating operations → structure: first reached for a **queue**, corrected to a **deque** once it was clear removals happen at *both* ends (dominate-and-drop off the back, evict-stale off the front). Then the load-bearing call: store **indices, not values** — because the "has the front fallen out of the window?" check is a question about position, and indices give the value back for free via `nums[q[0]]`. This is worth flagging since index-vs-value confusion is a recurring trap — got it right this time. Optimal was a clean first-try solve.

## Attempts

**Attempt 1 — brute force, broken bound**
```python
class Solution:
    def maxSlidingWindow(self, nums: List[int], k: int) -> List[int]:
        output = []
        l = 0
        r = k
        while r < len(nums):
            maximum = nums[l]
            for i in range(k):
                maximum = max(maximum, nums[l+i])
            output.append(maximum)
            r += 1
            l += 1
        return output
```

| Bug | Type | Explanation |
|-----|------|-------------|
| `while r < len(nums)` drops the last window | Implementation (off-by-one) | With `n` elements and window `k` there are `n-k+1` windows. This loop runs one short, so the final window (`l` at `n-k`) is never computed. Not an edge case — it fails on every non-trivial input. |
| Body dedented to column 0 | Syntax | `IndentationError` as pasted; likely a paste/tab artifact rather than the real editor state. |

**Breaking-input trace** (`nums=[1,3,-1,-3,5,3,6,7]`, `k=3`, original `r < 8`):

| l | window | max | r after | r<8? |
|---|--------|-----|---------|------|
| 0 | [1,3,-1] | 3 | 4 | yes |
| 1 | [3,-1,-3] | 3 | 5 | yes |
| 2 | [-1,-3,5] | 5 | 6 | yes |
| 3 | [-3,5,3] | 5 | 7 | yes |
| 4 | [5,3,6] | 6 | 8 | **no, stop** |

Output `[3,3,5,5,6]` — window `l=5 → [3,6,7] → 7` never reached. Fix: `r <= len(nums)`.

**Attempt 2 — brute force corrected.** Only the bound changed (`<` → `<=`). Logically correct, but O(n·k) → TLEs on LeetCode (`n` up to 10^5).

## Accepted Solution
```python
class Solution:
    def maxSlidingWindow(self, nums: List[int], k: int) -> List[int]:
        output = []
        q = collections.deque()  # holds indices
        l = r = 0

        while r < len(nums):
            # pop smaller values from the back (they're dominated)
            while q and nums[q[-1]] < nums[r]:
                q.pop()
            q.append(r)

            # front fell out of the window's left edge -> evict
            if l > q[0]:
                q.popleft()

            if (r + 1) >= k:
                output.append(nums[q[0]])
                l += 1
            r += 1

        return output
```
Result: Accepted, O(n). On LeetCode `collections` is available without an explicit import, but add `import collections` (or `from collections import deque`) if running anywhere else.

## Why It Works
**Invariant:** the deque holds indices whose corresponding values are strictly decreasing front → back, so `nums[q[0]]` is always the max of the current window.
**Key operation:** the back-domination pop — `while q and nums[q[-1]] < nums[r]: q.pop()` — discards every candidate that the incoming element makes permanently irrelevant.
**Non-obvious load-bearing line:** `if l > q[0]: q.popleft()`. `l` is reused as the current window's left edge (see Variables) — when the front index sits strictly left of it, the front is stale and gets dropped.

## How It Works
**Traversal:** `r` marches across every index once. The **back** of the deque absorbs new indices (after evicting dominated ones). The **front** is read for the answer and evicted when it expires. Each index is appended once and popped at most once → O(n) total despite the inner `while`.

**Business** (per step):
1. Evict dominated tail: pop back indices whose value `< nums[r]`.
2. Admit the new index: `q.append(r)`.
3. Evict stale front: if the front index is left of the window (`l > q[0]`), `popleft`.
4. Record once a full window exists (`r + 1 >= k`): append `nums[q[0]]` and advance `l`.

**Variables:**
| Variable | Represents | Stores | How It Changes | Why It Exists |
|----------|------------|--------|----------------|---------------|
| `output` | the answer | list of window maxes | one append per completed window | return value |
| `q` | live max candidates | **indices**, values strictly decreasing front→back | back pop on domination, front pop on expiry, append per step | O(1) max via `nums[q[0]]`; indices let you test expiry by position |
| `r` | right edge / current index | int | `+1` every iteration | drives the single pass |
| `l` | left edge of the current window | int | `+1` only when a window is recorded | doubles as "windows produced so far"; once windows start forming `l == r-k+1`, so `l > q[0]` is the expiry test |

> Note on `l`: it stays `0` until the first full window, so the expiry check can't fire early. After that it tracks the window's left boundary exactly. Clever, but subtle — the more common style computes the boundary inline as `r - k + 1`.

## Visual Walkthrough
`nums = [1,3,-1,-3,5,3,6,7]`, `k = 3`. Deque shown as indices; `→max` is `nums[q[0]]` when a window is recorded.

| r | nums[r] | after back-pop + append (q) | front evict? | window done? | output |
|---|---------|-----------------------------|--------------|--------------|--------|
| 0 | 1  | [0]        | no | no | — |
| 1 | 3  | [1]        | no | no | — |
| 2 | -1 | [1,2]      | no | yes → nums[1]=3 | [3] |
| 3 | -3 | [1,2,3]    | no | yes → nums[1]=3 | [3,3] |
| 4 | 5  | [4]        | no | yes → nums[4]=5 | [3,3,5] |
| 5 | 3  | [4,5]      | no | yes → nums[4]=5 | [3,3,5,5] |
| 6 | 6  | [6]        | no | yes → nums[6]=6 | [3,3,5,5,6] |
| 7 | 7  | [7]        | no | yes → nums[7]=7 | [3,3,5,5,6,7] |

(Front eviction never fires on this particular input because domination clears the deque first — a window like `[1,4,2]` with the max on the *left* edge is what exercises `popleft`.)

## Optimization Journey
1. **Brute force:** rescan each window for its max — O(n·k) time, O(1) extra.
2. **Bottleneck:** every window recomputes from scratch, discarding all knowledge of the previous window's contents.
3. **Observation:** when a new element arrives, every smaller element to its left is permanently dominated — any future window containing them also contains the bigger, later-expiring element.
4. **Optimization:** keep a monotonic (strictly decreasing) deque of indices; max is always at the front; drop dominated indices off the back, expired indices off the front.
5. **Final:** an interviewer reaches this by noticing the rescan is redundant work, then asking "what's the cheapest structure that gives me the window max in O(1) and survives a slide?" → a deque maintained in decreasing order. O(n) amortized.

## Pattern Recognition
**Signals:** "maximum/minimum of every fixed-size window" → monotonic deque. "Fixed window, one answer per position" → sliding window family. Large `n` constraint (10^5) → the naive O(n·k) won't pass, so the deque is mandatory not optional.

**Pattern used:** Monotonic Deque (within the Sliding Window family) — gives O(1) window-max while sliding, in amortized O(n).

| Pattern | Why It Fits | Why It Doesn't | Verdict |
|---------|-------------|----------------|---------|
| Sliding Window (brute) | fixed window, slide one at a time | O(n·k) rescan → TLE | correct logic, too slow |
| Heap (max-heap) | repeatedly grab the max of a changing set | O(n log n); needs lazy deletion of stale indices | works, but slower + fiddlier |
| Monotonic Deque | O(1) front max, both-end maintenance, amortized O(n) | more machinery to reason about | **optimal** |

## Edge Cases & Debugging
| Case | Input | Expected | How Handled |
|------|-------|----------|-------------|
| last window dropped | any `n>k` | full `n-k+1` outputs | the original `r < len` bug; fixed bound, then deque records on `r+1 >= k` |
| `k == 1` | `[a,b,c]`, k=1 | `[a,b,c]` | each element is its own window; deque holds one index, front == current |
| `k == len(nums)` | whole array, k=n | one max | single window; deque collapses to the global max's index |
| max on the left edge | e.g. `[4,1,2]` | front must expire correctly | `if l > q[0]: q.popleft()` evicts the stale front |

**Recurring-bug note:** the original mistake was an **off-by-one on the loop bound** — same off-by-one family that bites you on slice bounds. The flip side: you nailed the **index-vs-value** decision here (store indices), which is the recurring confusion you *usually* trip on. Good rep.

## Complexity
| Metric | Complexity | Reason |
|--------|------------|--------|
| Time   | O(n)       | single pass; each index appended once and popped at most once (amortized) — the inner `while` is not a second factor |
| Space  | O(k)       | the deque holds at most `k` indices; `output` is the required result, not counted as auxiliary |

## Interview Prep
**30s:** Sliding-window max → monotonic decreasing deque of indices, front is always the window max. Drop dominated indices off the back, expired ones off the front. O(n) time, O(k) space.

**2 min:** Signal is "max of every fixed window." Brute force rescans each window for O(n·k), which TLEs at n=10^5. Key insight: when a new element arrives, every smaller element to its left can never be a max again, so maintain a deque of indices with strictly decreasing values. Invariant: front index is the current window's max. Each step pops dominated tail indices, appends the new one, evicts the front if it left the window, and records `nums[front]` once a full window exists. Each index enters and leaves once, so it's amortized O(n). Tradeoff vs a heap: heap is O(n log n) and needs lazy deletion of stale entries; the deque avoids both.

**Follow-ups:**
- *Why indices, not values?* Expiry is a position question (`is front < window's left edge`); indices answer it and still give the value via `nums[idx]`.
- *Window minimum instead?* Same structure, flip the comparator to keep an increasing deque.
- *Why is the inner `while` not O(n²)?* Amortized — total pops ≤ total appends = n.
- *Streaming version?* Same deque works online; you emit a max each time the window is full.

## Revision
**1 week:** restate the invariant (front index = window max, values strictly decreasing front→back) and re-trace an input where the max sits on the **left** edge so the `popleft` expiry actually fires (e.g. `[4,1,2,...]`).
**1 month:** skeleton — `while r: pop dominated back; append r; evict stale front; record when window full`. Complexity reason: each index in/out once → O(n).
**Mental model:** a bouncer line where the strongest person stands at the front. Anyone weaker who walks in behind a stronger person already inside can't ever be "the strongest remaining," so they're sent home immediately (back eviction). The front person leaves only when they age out of the window (front eviction). At any moment the front of the line is your answer.
**Takeaways:** when you need the extreme of a sliding window in O(1), reach for a monotonic deque of indices; "store position not value" whenever expiry is positional; an inner loop bounded by total insertions is amortized linear, not quadratic.
