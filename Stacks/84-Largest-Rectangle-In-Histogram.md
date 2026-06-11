# 84. Largest Rectangle In Histogram

## Problem

Given an array `heights` where each value is the height of a bar in a histogram (all bars width 1, sitting side by side), find the area of the largest rectangle that can be drawn **inside** the histogram. The rectangle must be solid (no gaps) and its height is capped by the shortest bar it spans.

**Example:** `heights = [2,1,5,6,2,3]` → answer `10` (the height-5 bar and height-6 bar form a 5-tall, 2-wide rectangle).

```
6:           █
5:        █  █
4:        █  █
3:        █  █        █
2: █      █  █     █  █
1: █  █   █  █     █  █
   2  1   5  6     2  3
  i0 i1  i2 i3    i4 i5

   biggest rectangle: height 5 × width 2 = 10
   (spans i2 and i3 — the 6 gets "lowered" to 5 to keep the rectangle solid)
```

---

# My Learning Journey

This one was a fight, and the value is in *where* it broke, not that it eventually passed.

**The starting misconception.** First attempt wasn't even modeling the problem — it merged adjacent equal-height bars by summing them and returned the max value. The mental model was "a rectangle is equal-height bars stacked together." The real model never clicked until the trade got spelled out: **a rectangle's height is capped by its shortest bar, so you trade height for width.** A short bar can make a wide rectangle, a tall bar a skinny one.

**The indentation false-alarm.** An early version looked like it dropped the first bar (`heights[0]`) because the push lived inside `if i >= 1`. Turned out the paste indentation was misread — a screenshot showed an `else` branch did push i0. Not a real bug, walked it back.

**Getting to brute force.** Once "each bar gets to be the shortest one, find how wide it can stretch" landed, brute force came together: for each bar, spread left and right, stop at the first shorter bar, multiply height × width, track the max. Two real bugs got caught here — an **IndexError** on the right walk (`heights[j]` before bounds-checking `j`) and a **silent negative-index wraparound** on the left walk (`heights[-1]` doesn't crash in Python, it grabs the wrong end). Fix was putting the bounds check *first* in each `while` so Python's `and` short-circuits before touching the index. Brute force passed at O(n²).

**The monotonic stack grind.** This is where most of the chat went. The stack version got rebuilt across many attempts, each killing one bug:

- **Width as a pop-count.** Early tries did `width += 1` per pop. This *can't* work — popped-out bars leave gaps that earlier bars stretch across, and counting pops can't see those gaps. This was the core misunderstanding and it took the longest to shake.
- **Wrong height in the area calc.** Multiplied by `heights[i]` (the current/triggering bar) instead of `heights[popped]` (the bar that owns the rectangle).
- **`while stack:` drained the whole stack** instead of only popping bars taller than current — destroying the left wall.
- **Area + max update trapped inside the `else`** so the empty-stack case computed a width but never recorded it — silently dropping the biggest (full-reach) rectangles.
- **`n` never defined**, NameError in the drain.
- **Push-before-pop** with a conditional push — on descending inputs like `[4,3]` a bar got popped but never pushed, vanishing entirely and returning 4 instead of 6.

**The fix that finally passed:** pop-first, push unconditionally at the bottom of the loop, area/max pulled out of the `else`, empty-stack width branch, and an end-drain. `[4,3]` returned 6, accepted.

**The lingering confusion (resolved last).** Even after it passed, the `-1` in `width = i - stack[-1] - 1` wasn't understood — it was typed because it was suggested, not because it was owned. That got broken down via fencepost counting (how many bars sit *strictly between* two walls). Then NeetCode's solution got pasted, which has **no `-1`** — and the final realization was that NeetCode stores the rectangle's left *edge* (`start`) instead of the wall, so its `i - index` and the textbook `i - stack[-1] - 1` produce the identical number.

---

# Attempt 1 — Merge Equal Bars (Wrong: Logic)

```python
class Solution:
    def largestRectangleArea(self, heights: List[int]) -> int:
        stack=[]
        for i in range(len(heights)):
            if i >= 1 :
                if heights[i] == heights[i-1]:
                    temp=stack.pop()
                    stack.append(temp+heights[i])
                else:
                    stack.append(heights[i])
            else:
                stack.append(heights[i])
        stack.sort()
        return (stack.pop())
```

**What I was thinking:** a rectangle is made of equal-height adjacent bars, so merge them and take the biggest pile.

**Why it's wrong:** never models width at all. On `[2,1,5,6,2,3]` no two adjacent bars are equal, so the merge branch never fires — it just returns `max(heights)` = 6. Answer is 10. The whole frame (height = a stack of equal bars) is the wrong mental model.

---

# Attempt 2 — Brute Force (Accepted, O(n²))

```python
class Solution:
    def largestRectangleArea(self, heights: List[int]) -> int:
        stack=[]
        for i in range(len(heights)):
            height=heights[i]
            width=1
            j=i+1
            while j < len(heights) and heights[i] <= heights[j]:   # right walk
                j +=1
                width +=1
            j=i-1
            while j >= 0 and heights[i] <= heights[j]:             # left walk
                j -=1
                width +=1
            area=height * width
            stack.append(area)
        stack.sort()
        return stack[-1]
```

**The bugs that got fixed to reach this:**
- Right walk crashed with IndexError because `heights[j]` was checked before `j < len(heights)`. Fix: bounds check first (Python short-circuits `and`).
- Left walk silently wrapped: `heights[-1]` grabs the last element instead of crashing. Fix: `j >= 0` first.

Passes, but re-scans the same bars repeatedly to find walls → O(n²). The `sort()` + `[-1]` is a roundabout `max()`.

---

# Accepted Solution — Monotonic Stack (O(n))

```python
class Solution:
    def largestRectangleArea(self, heights: List[int]) -> int:
        stack=[]                                              # holds INDICES, heights increasing
        maxArea=0
        for i in range(len(heights)):
            while stack and heights[stack[-1]] > heights[i]:  # pop strictly-taller bars
                popped=stack.pop()
                if not stack:
                    width=i                                   # no left wall → reaches start
                else:
                    width=i-stack[-1]-1                       # gap between the two walls
                area=heights[popped]*width
                if area > maxArea:
                    maxArea=area
            stack.append(i)                                   # ALWAYS push, after the while
        n=len(heights)
        while stack:                                          # drain leftovers → right wall = n
            popped=stack.pop()
            if not stack:
                width=n
            else:
                width=n-stack[-1]-1
            area=heights[popped]*width
            if area > maxArea:
                maxArea=area
        return maxArea
```

---

# Why It Works

**The reframe that unlocks everything:** every rectangle in the histogram has *some* bar that is its shortest. So instead of trying every rectangle (too many), let **each bar take a turn being "the shortest bar"** and ask: "if I'm the floor, how wide can my rectangle go?" Width spreads left and right until it hits a bar shorter than me (a shorter bar can't hold my height — it lowers the roof). Check every bar as the floor, take the max. You can't miss the answer because the real answer's height is pinned to one of the bars.

**The invariant (what makes the stack O(n)):** the stack always holds indices whose heights are **strictly increasing** bottom → top. That ordering guarantees both walls are known the instant you pop a bar:

- the bar that triggered the pop (current `i`) is shorter → it's the **right wall**
- whatever's left underneath on the stack is shorter → it's the **left wall**

No re-scanning. The width is just the gap between those two walls.

---

# Traversal Logic

*How we move through the data.*

- Single left-to-right pass over the indices via `for i in range(n)`.
- Inside each step, a `while` pops zero or more bars off the top of the stack.
- The stack itself is the second axis of movement: indices get **pushed** (bar enters the "waiting room") and **popped** (bar's right wall arrived, finalize it).
- A final drain pass pops everything still waiting.

Each index is pushed exactly once and popped exactly once → 2n stack operations total → O(n). The nested `while` does **not** make it quadratic (see Complexity Analysis).

---

# Business Logic

*The decision made at each step.*

- **Pop condition:** `heights[stack[-1]] > heights[i]` — pop only bars **strictly taller** than the current bar. Equal bars stay (they get pushed and resolved later). The current short bar is their right wall.
- **Width decision:** if the stack is empty after popping, the rectangle reaches the start → `width = i`. Otherwise it's bounded on both sides → `width = i - stack[-1] - 1`.
- **Area + max update:** computed **per popped bar**, using `heights[popped]` (never `heights[i]`).
- **Push decision:** unconditional, at the bottom of every iteration. Every bar enters the stack after it's done evicting taller bars.
- **Drain decision:** leftover bars never met a shorter bar on their right, so their right wall is the end → `width = n - stack[-1] - 1` (or `n` if empty).

---

# The Bouncer Analogy (Mental Model)

Bars walk into a club one at a time, left to right. There's a line inside with one rule: **the line must go short → tall, front to back.**

- **New bar is taller** than the back of the line → he gets in line and waits. We don't know how far right he reaches yet.
- **New bar is shorter** than the back → he'd break the rule. The bouncer kicks out everyone taller than him first. **The instant a bar is kicked out, both his walls are known:** the new short bar is his right wall, and whoever's still in line behind him is his left wall (the line is sorted, so they're shorter). Compute his rectangle on the way out.

```
line (short→tall):  [1, 5, 6]
bar 2 arrives:
  kick 6 → right wall = 2, left wall = 5 (behind it). width=1, area=6
  kick 5 → right wall = 2, left wall = 1 (behind it). width=2, area=10  ← 5 stretches
           over where 6 used to stand, because 6 was taller
  1 is shorter than 2 → stop kicking, bar 2 gets in line
```

stack = the line. push = get in line. pop = your wall arrived, here's your rectangle.

---

# Variable Roles

| Variable | Represents | Stores | How It Changes | Why It Exists |
|----------|------------|--------|----------------|---------------|
| `stack` | bars still waiting for their right wall | list of **indices** (not heights) | push when a bar enters, pop when its right wall (a shorter bar) appears | remembers boundaries so we never re-scan for walls |
| `i` | current bar's position / the right wall | int index | increments each outer loop | the shorter bar that closes out taller bars on its left |
| `maxArea` | best rectangle seen so far | int | updated on every pop if a bigger area is found | the answer accumulator |
| `popped` | the bar being finalized right now | int index | reassigned each pop | the bar whose rectangle we're computing (height = `heights[popped]`) |
| `width` | how wide the popped bar's rectangle is | int | recomputed per pop | bars strictly between left wall and right wall |
| `n` | total number of bars | int | set once | the right wall for the drain pass |

---

# THE `-1` EXPLAINED (the part that was confusing)

This is pure counting. Forget code.

After you pop a bar, there's a **left wall** (`stack[-1]`) and a **right wall** (`i`). The rectangle covers the bars **strictly between** them — the walls themselves are NOT inside it (they're shorter, they're the fence, not the field).

**How many positions are strictly between index L and index R?**

```
index:   1   2   3   4   5
         L   a   b   c   R
             └───┬───┘
            bars between = a, b, c = 3
```

Get 3 from the numbers 1 and 5:

```
R - L     = 5 - 1     = 4   ← too big by one
R - L - 1 = 5 - 1 - 1 = 3   ← correct
```

**Why `R - L` overcounts by 1 — the fencepost:**

```
   L   a   b   c   R
    \_/ \_/ \_/ \_/
     1   2   3   4      ← R - L = 4 STEPS (gaps), not bars
```

`R - L` counts *steps* from L to R, and the last step lands on R — which is a wall, not a middle bar. Subtract 1 to drop it. So `width = R - L - 1 = i - stack[-1] - 1`.

**Concrete, from `[2,1,5,6,2,3]` (pop-i2 moment, stack `[1]`, i=4):**

```
index:   1   2   3   4
         1   5   6   2
         L  [a] [b]  R
         ↑           ↑
     left wall    right wall

   bars between index 1 and 4 = index 2 and 3 = the 5 and the 6 = 2 bars
   width = i - stack[-1] - 1 = 4 - 1 - 1 = 2   ✓
```

**Why empty-stack uses `width = i` (no `-1`):** when the stack empties after popping, there's **no left wall** — the rectangle reaches all the way back to index 0. Bars sit at index 0, 1, ..., i-1, which is exactly `i` bars. No left wall to subtract, so no `-1`.

```
index:  0   1   2   ... i-1   i
        a   b   c    ...  d    R
                              ↑ right wall, no left wall
        bars 0..i-1 = i bars  →  width = i
```

**One sentence to remember:** *the `-1` is there because the two walls aren't part of the rectangle — you're counting the bars trapped between them, and "between" always has one fewer than the raw distance.*

---

# Visual Walkthrough (Full Trace)

Input `[2,1,5,6,2,3]`, expected `10`.

| i | h | stack before | action | popped (h) | width | area | maxArea |
|---|---|--------------|--------|-----------|-------|------|---------|
| 0 | 2 | `[]` | push | — | — | — | 0 |
| 1 | 1 | `[0]` | pop i0, push | 2 | `i`=1 (empty) | 2 | 2 |
| 2 | 5 | `[1]` | push | — | — | — | 2 |
| 3 | 6 | `[1,2]` | push | — | — | — | 2 |
| 4 | 2 | `[1,2,3]` | pop i3 | 6 | `4-2-1`=1 | 6 | 6 |
|   |   | `[1,2]` | pop i2 | 5 | `4-1-1`=2 | **10** | **10** |
|   |   | `[1]` | push (1<2 stop) | — | — | — | 10 |
| 5 | 3 | `[1,4]` | push | — | — | — | 10 |
| **drain** | | `[1,4,5]` | pop i5 | 3 | `6-4-1`=1 | 3 | 10 |
|   |   | `[1,4]` | pop i4 | 2 | `6-1-1`=4 | 8 | 10 |
|   |   | `[1]` | pop i1 | 1 | `n`=6 (empty) | 6 | 10 |

Final: **10**.

ASCII of the winning moment (i=4 pops i2, the height-5 bar stretching across where 6 was):

```
        ┌─────┐
6:      │  ░  │ ← the 6 already popped; 5 fills its column
5:      │█ █ █│  wait — width is i2..i3 = 2 columns, height 5
4:      │█ █ █│
3:      │█ █ █│
2: █  ░ │█ █ █│ ░  ░
1: █  ░ │█ █ █│ ░  ░
   2  1  5  6  2  3
        └ 5×2 = 10 ┘
```

---

# Optimization Journey

1. **Brute Force:** for each bar, spread left + right until a shorter bar, `height × width`. O(n²).
2. **Bottleneck:** every bar re-walks its neighborhood to *find* its walls, re-scanning bars previous bars already scanned. Flat input `[3,3,3,3]` = every bar walks the whole array.
3. **Observation:** as you read left to right, a bar can't know its right wall until a shorter bar appears. So keep bars "waiting" in a structure, sorted short→tall. The moment a shorter bar shows up, it's the right wall for everyone taller — and the left wall is whoever's still waiting behind them. **Both walls handed to you on pop, zero re-scan.**
4. **Optimization:** monotonic (increasing) stack of indices. Push while climbing, pop+compute on the drop.
5. **Final Approach:** an interviewer discovers this by noticing the brute-force redundancy ("I keep re-finding the same boundaries") and asking "can I remember boundaries instead of recomputing them?" → a stack that maintains the increasing invariant.

---

# Pattern Recognition

## Signal Phrases
- "largest rectangle / max area" under bars or in a histogram
- each element needs to know "how far can I extend before hitting something smaller"
- "previous/next smaller element" shape (boundaries defined by a shorter neighbor)

## Pattern Used
**Monotonic Stack** (increasing).

## Why This Pattern Fits
- The answer for each bar depends on its nearest-shorter-bar on both sides — the textbook monotonic stack job.
- One left-to-right pass with push/pop resolves all boundaries in O(n).
- The "pop when the current element breaks the monotonic order" mechanic maps directly onto "right wall found."

---

# Pattern Comparison

| Pattern | Why It Fits | Why It Doesn't | Verdict |
|---------|-------------|----------------|---------|
| Monotonic Stack | boundaries = nearest shorter bar; resolves in O(n) | — | **Use this** |
| Brute Force | correct, easy to reason about | O(n²), re-scans | Fallback / stepping stone |
| Divide & Conquer | recurse on the min bar, split left/right | O(n log n) avg, O(n²) worst, awkward | Real alt, not optimal |
| Two Pointers | tempting ("expand outward") | no clean single shortest-bar invariant; can't place two pointers meaningfully | Red herring |

---

# Common Mistakes

| Mistake | What actually happened | Fix |
|---------|------------------------|-----|
| Width as a count of pops (`width += 1`) | can't see gaps left by already-popped bars; gives 1 when real width is 2 | width = `i - stack[-1] - 1`, index math |
| `area = heights[i] * width` | multiplied by the triggering bar, not the owner | `heights[popped] * width` |
| `while stack:` drains everything on one drop | destroys the left wall | `while stack and heights[stack[-1]] > heights[i]` |
| area/max trapped in `else` | empty-stack rectangles (the biggest ones) never recorded | dedent area + max-check out of `else` |
| push-before-pop, conditional push | bars vanish on descending input (`[4,3]`→4 not 6) | pop first, then `stack.append(i)` unconditionally at loop bottom |
| right-walk IndexError / left-walk wraparound (brute force) | `heights[j]` before bounds check | bounds check first, rely on `and` short-circuit |
| forgot the drain | leftover bars never computed | second `while stack` with right wall = `n` |
| named accumulator `max` | shadows the builtin | `maxArea` |

---

# Edge Cases

| Case | Input | Expected | How Handled |
|------|-------|----------|-------------|
| Strictly descending | `[4,3]` | 6 | each pop+drain; unconditional push keeps every bar alive |
| Strictly increasing | `[1,2,3,4]` | 6 | nothing pops until the drain; all reach the end |
| All equal | `[3,3,3]` | 9 | `>` (not `>=`) keeps equals on the stack; drain resolves full width |
| Single bar | `[5]` | 5 | no pops in loop; drain pops it with `width=n=1` |
| Valley | `[5,1,5]` | 5 | the 1 walls off both 5s; neither stretches across |

---

# Debugging Lessons

- **Bounds check before index access**, and rely on Python's left-to-right `and` short-circuit. This killed both the IndexError (right walk) and the silent negative-wraparound (left walk) in brute force.
- **Indentation is logic in Python.** Two correct lines (`area`, `if area > maxArea`) trapped one level too deep inside an `else` silently dropped the biggest rectangles. The algorithm was right; the block structure wasn't.
- **Order of operations matters:** pop *then* read `stack[-1]` for the wall — reading before popping reads the bar itself, not its wall.
- **Pop before push, push unconditionally.** Conditional pushes leak bars on monotonic inputs.

---

# NeetCode / Official Solution

```python
class Solution:
    def largestRectangleArea(self, heights: List[int]) -> int:
        maxArea = 0
        stack = []  # pair: (index, height)

        for i, h in enumerate(heights):
            start = i
            while stack and stack[-1][1] > h:
                index, height = stack.pop()
                maxArea = max(maxArea, height * (i - index))
                start = index
            stack.append((start, h))

        for i, h in stack:
            maxArea = max(maxArea, h * (len(heights) - i))
        return maxArea
```

## Comparison to My Solution

Same algorithm, different bookkeeping. The headline difference: **NeetCode has no `-1`.**

| | My version | NeetCode |
|---|---|---|
| Stack holds | indices | `(start, height)` pairs |
| Width formula | `i - stack[-1] - 1` | `i - index` |
| The `-1` | explicit fencepost correction | baked into `start` |
| Left boundary | derived from `stack[-1]` on pop | carried inside the tuple |

**Why NeetCode drops the `-1`:** when a bar is popped, the current bar can extend left into the popped bar's space (the popped bar was taller). So `start = index` drags the current bar's starting point leftward each pop, and it pushes with that *earliest* start. The stored `index` is therefore the **left edge of the rectangle**, not the wall. So `i - index` is a direct width with no correction needed.

The relationship: **NeetCode's stored `index` = my `stack[-1] + 1`.** That `+1` (storing the edge instead of the wall) is exactly where my `-1` went. They cancel into the identical number.

Proof on `[2,1,5,6,2,3]`, the i=4 pop of the height-5 bar:
- Mine: `width = i - stack[-1] - 1 = 4 - 1 - 1 = 2`
- NeetCode: pops `(2,5)`, `width = i - index = 4 - 2 = 2`

Same 2, same area 10.

**Verdict:** keep mine. The explicit `-1` makes wall-exclusion visible and is easier to defend in an interview than NeetCode's `start = index` trick, which reads as "wait, why does that work."

---

# Deep Understanding

The non-obvious line is `width = i - stack[-1] - 1`, and the non-obvious fact is that **a popped bar's width can exceed the number of bars popped before it.** When the height-5 bar pops, width is 2 even though only the height-6 bar was popped before it — because the 6 was *taller* than the 5, so the 5 happily stretches across the column the 6 vacated. The stack "remembers" the 5's true left wall is way back at index 1. This is the single insight the entire pattern hinges on, and it's why width must be index-math, never a pop-count.

---

# Complexity Analysis

| Metric | Complexity | Reason |
|--------|------------|--------|
| Time | O(n) | each index pushed once and popped once → ≤ 2n stack ops. The nested `while` is **amortized O(1)** per outer step — total pops across the whole run ≤ n |
| Space | O(n) | increasing input (`[1,2,3,4]`) pushes all n indices before any pop |

**The interview trap:** the `while` inside the `for` looks O(n²). It isn't — each bar can only be popped once ever, so the inner loop's total work across all iterations is bounded by n. Say "amortized" out loud and you've shown you understand it.

---

# Related Problems

| # | Problem | Difficulty | Same Pattern? | Notes |
|---|---------|------------|---------------|-------|
| 739 | Daily Temperatures | Medium | Yes (monotonic stack) | Gentler width logic; the better *first* problem to learn this on |
| 496 | Next Greater Element I | Easy | Yes | Baseline "next greater/smaller via stack" |
| 503 | Next Greater Element II | Medium | Yes | Circular array twist |
| 85 | Maximal Rectangle | Hard | Yes — *calls this exact function* per matrix row | Direct parent-child of 84 |
| 42 | Trapping Rain Water | Hard | Related | Stack or two-pointer, similar boundary thinking |

---

# Interview Explanation (30 Seconds)

Every rectangle's height is set by its shortest bar, so I let each bar be the shortest and find how wide it can stretch before hitting a shorter bar on either side. A monotonic increasing stack of indices finds both boundaries in one pass — when a shorter bar appears, it's the right wall for everything taller on the stack, and the left wall is whatever's underneath. O(n) time, O(n) space.

---

# Interview Explanation (2 Minutes)

The recognition signal is "max rectangle where each element's reach depends on its nearest *smaller* neighbor on both sides" — that's a monotonic stack. Brute force is O(n²): for each bar, expand both directions until a shorter bar, multiply height × width. The bottleneck is re-scanning the same bars to re-find boundaries.

The optimization: keep a stack of indices whose heights strictly increase. While bars climb, push and wait — you don't yet know any bar's right boundary. The moment a shorter bar arrives, it *is* the right boundary for every taller bar on the stack, so you pop each one and finalize it. The left boundary is free: it's whatever's left underneath after popping, which is shorter by the invariant. Width is `i - stack[-1] - 1` (the bars strictly between the two walls — the `-1` excludes the walls themselves), or `i` if the stack emptied (reaches the start). After the loop, drain anything still on the stack with the right wall set to `n`. The invariant guarantees each index is pushed and popped once, so it's O(n) — the nested loop is amortized, not quadratic. The one tradeoff vs. divide-and-conquer (O(n log n)) is O(n) extra space for the stack.

---

# Common Follow-Up Questions

- **"Isn't the nested loop O(n²)?"** No — amortized O(n). Each index is popped at most once across the entire run, so total inner-loop work is bounded by n.
- **"Why `>` and not `>=` in the pop condition?"** Equal-height bars are kept on the stack and resolved later; using `>` avoids double-handling and keeps the width math clean. `>=` also produces the correct max but pops equals eagerly.
- **"Can you do it without the drain?"** Yes — append a sentinel `0` to `heights`; it forces every real bar to pop before the loop ends. Cleaner but less obvious.
- **"How would you extend this to a 2D matrix?"** That's LC 85 — build a histogram per row (running column heights) and call this function on each row.
- **"Space optimization?"** Stack is O(n) worst case and unavoidable for the one-pass version; divide-and-conquer trades it for O(log n) recursion depth but worse time.

---

# Key Takeaways

- **Width is the gap between walls, computed with index math — never a count of pops.** Popped bars leave gaps earlier bars stretch across; only `i - stack[-1] - 1` sees them.
- **The `-1` excludes the two walls** (they're shorter, they cap the rectangle, they're not inside it). Empty stack → no left wall → no `-1`, width = `i`.
- **Pop before push; push unconditionally.** The clean shape for every monotonic-stack problem: `while (pop condition): pop+process`, then `push`.
- **The nested loop is amortized O(n)** — name it in interviews.

---

# Revision Notes

**1 week:** rebuild from scratch, blank file. If you stall, it'll be on the width formula or the pop-before-push order — those were the two hardest. Re-derive the `-1` with the fencepost picture (bars strictly between index L and R = `R - L - 1`).

**1 month:** you should only need the bouncer analogy + "width = gap between walls, index math not pop-count" to reconstruct the whole thing. If `i - stack[-1] - 1` looks foreign again, redo the fencepost trace on `[2,1,5,6,2,3]`.

---

# Final Mental Model

It's a club with a velvet rope and a strict bouncer. Bars line up, and the line is always shortest-to-tallest front to back. A bar waits in line because it doesn't yet know how far right its rectangle reaches. The instant a shorter bar walks up, that short bar is the right wall for everyone taller — so the bouncer evicts them one by one, and as each leaves, the person still standing behind them is automatically their left wall. You measure the floor space between those two walls (and the `-1` is just "don't count the walls themselves, only the room between them"), multiply by the evicted bar's height, and that's its biggest possible rectangle. Check everyone who ever gets evicted — plus everyone still in line at closing time, who reach all the way to the back wall — and the largest floor space anyone claimed is your answer.
