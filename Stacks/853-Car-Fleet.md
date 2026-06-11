# 853. Car Fleet

## Problem

`n` cars drive toward the same destination on a one-lane road. You get `position[i]` (how far along the road car `i` already is) and `speed[i]`. The destination is at `target` miles. A car can't pass the car ahead — it can only catch up and then match that car's speed. A **fleet** is one or more cars arriving at the same position and speed (a lone car counts as a fleet). If a car reaches a fleet exactly at the destination, it joins. Return the number of distinct fleets that arrive.

Concrete example:

```
target = 10, position = [4,1,0,7], speed = [2,2,1,1]
→ 3 fleets
```

Cars at 4 and 7 merge into one fleet at the line. Cars at 1 and 0 never catch the car ahead. That's 3 fleets.

---

# My Learning Journey

This one was a multi-pass grind, and the bugs were almost all in the *counting* and *initialization*, not the core idea.

**First crack at the physics.** I initially wrote the formula pointed the wrong way — something like `position[i] + speed[i] * t <= target`, which is a "where is the car at time t" equation. The realization: I don't care where a car is at an arbitrary moment, I only care *when* it reaches the target. Solving `position[i] + speed[i]*t = target` for `t` gave me the one number per car that the whole problem hinges on: `t = (target - position[i]) / speed[i]`.

**The set dead-end.** My first instinct was to dump every car's `t` into a set and return the count of unique times. It happened to give 3 on example 2 — but that was a coincidence. Two cars with equal times aren't automatically a fleet, and two cars with different times *can* end up in the same fleet (faster car gets bottlenecked behind a slower one ahead). So distinct-time-counting is wrong. The comparison has to be against *the car ahead*, not against all times globally.

**Position confusion.** I got stuck on whether `position[i]` meant miles remaining or miles traveled. Cleared it up: it's distance from the start (mile 0), so bigger position = further along = closer to the target = *ahead*. Miles remaining is `target - position[i]`.

**Direction insight.** The car with the biggest position has nothing ahead of it — its fate is locked, it cruises in at its own time. That's the car whose outcome I know for certain first. So I sort by position ascending, then scan **front to back** (biggest position first). Processing in that direction means the fleet ahead is always already resolved by the time I reach the next car behind.

**The bug parade (this is where most of the work was):**
- Attempt 1–2: comparison was backwards (`t1 >= t2` with `t1` being the front car), pairwise stack comparison that loses track of chained merges, and a `stack[-2]` access that `IndexError`s on a one-element stack.
- Attempt 3: condition flipped the wrong way — I was subtracting from the fleet count *and* updating `cur` on the merge case, when both should happen on the *new-fleet* case.
- Attempt 4: fixed the condition to `t > cur` and switched to counting up from 0, but two bugs survived — I initialized `cur` to the front car's time (which makes the first car compare against itself and never count), and I had `cur = t` *outside* the `if`, so merging cars wrongly lowered the reference time.
- Attempt 5 (accepted): initialized `cur = 0` so the first car always counts, and moved `cur = t` *inside* the `if`. Landed on 3.

The thing that finally exposed each bug was hand-tracing example 2 and watching it print `fleet = 2` instead of `3`.

---

# Attempt 1

```python
class Solution:
    def carFleet(self, target, position, speed):
        stack = []
        fleet = len(speed)
        for i in range(len(speed)):
            t = (target - position[i]) / speed[i]
            stack.append(position[i], t)      # bug: append takes ONE arg
        stack.sort                            # bug: not called, just referenced
        while stack:
            p1, t1 = stack[-1]
            p2, t2 = stack[-2]                # bug: IndexError when 1 element left
            if (t1 >= t2):                    # bug: comparison direction backwards
                stack.pop()
                fleet -= 1
            else:
                stack.pop()
        return fleet
```

What I was thinking: pair position with time, sort, walk the stack comparing the top two cars, reduce a fleet count seeded at `len(speed)` whenever two merge.

What went wrong: `append` got two args instead of a tuple, `sort` was never called, comparing `stack[-1]` vs `stack[-2]` is pairwise (loses chained fleets), the inequality was mirrored, and the last iteration accesses `stack[-2]` on a single-element list → crash.

# Attempt 2

Fixed only the append to `stack.append((position[i], t))`. Everything else unchanged, so all other bugs remained.

# Attempt 3

```python
        stack.sort()
        temp, cur = stack[-1]                 # bug: init cur to front car's time
        while stack:
            p, t = stack[-1]
            if (t <= cur):                    # bug: merge case shouldn't subtract/update
                stack.pop()
                fleet -= 1
                cur = t
            else:
                stack.pop()
        return fleet
```

Introduced the `cur` scalar. But the logic was inverted — subtracting and updating `cur` on the merge case (`t <= cur`) instead of on the new-fleet case. Traced example 2 → returned `2`.

# Attempt 4

```python
        fleet = 0
        stack.sort()
        temp, cur = stack[-1]                 # bug: still init to front car
        while stack:
            p, t = stack.pop()
            if (t > cur):
                fleet += 1
            cur = t                           # bug: outside the if
        return fleet
```

Switched to counting up from 0 with the correct `t > cur` condition. But `cur` still initialized to the front car (so car #1 compares against itself and never counts), and `cur = t` ran every iteration (so a merging car lowered the reference). Traced example 2 → returned `2`.

---

# Accepted Solution(s)

```python
class Solution:
    def carFleet(self, target: int, position: List[int], speed: List[int]) -> int:
        stack = []
        fleet = 0
        for i in range(len(speed)):
            t = (target - position[i]) / speed[i]   # solo time to reach target
            stack.append((position[i], t))
        stack.sort()                                # ascending by position
        cur = 0                                     # below any positive time
        while stack:
            p, t = stack.pop()                      # front-to-back (biggest pos first)
            if (t > cur):                           # can't catch fleet ahead → new fleet
                fleet += 1
                cur = t                             # this fleet is now the slowest ahead
        return fleet
```

---

# Why It Works

Each car's solo arrival time `t = (target - position) / speed` is the time it'd take to reach the destination if nothing blocked it. Sorting by position and scanning from the car closest to the target backward means the fleet ahead is always fully resolved before I reach the next car. `cur` holds the arrival time of the slowest fleet currently ahead. A car can only be blocked by something in front, so:

- If a car's solo `t > cur`, nothing ahead is slow enough to stop it being its own, slower fleet — it becomes the new bottleneck.
- If `t <= cur`, it would arrive sooner unobstructed, meaning it rams the back of the slower fleet ahead and inherits that fleet's pace — no new fleet.

The invariant: `cur` only ever increases, and it always equals the arrival time of the slowest surviving leader ahead.

---

# Traversal Logic

How we move through the data:

- Build a list of `(position, t)` pairs — one linear pass over the input.
- `sort()` orders pairs ascending by position (first tuple element).
- The `while stack: stack.pop()` walks the list **from the end**, i.e. biggest position → smallest position. That's front-of-the-road → back-of-the-road.

No two pointers, no nested traversal. One sort plus one linear back-to-front scan.

---

# Business Logic

The decision made at each car:

- Compute nothing new mid-scan — the `t` is already stored.
- Compare the current car's `t` against `cur`.
- **Decision:** `t > cur` → this is a new fleet → increment `fleet`, update `cur = t`. Otherwise → it merges → do nothing.

The only state mutated by a decision is `fleet` (the answer) and `cur` (the reference), and both change only on the new-fleet branch.

---

# Variable Roles

| Variable | Represents | Stores | How It Changes | Why It Exists |
|----------|------------|--------|----------------|---------------|
| `stack`  | all cars as (position, time) pairs | list of `(int, float)` tuples | appended once per car, sorted, popped during scan | holds the cars in road order so we can process front-to-back |
| `t`      | a car's solo time to reach target | float | recomputed per car in build loop, reassigned per pop in scan | the single quantity catch-up depends on |
| `fleet`  | running count of distinct fleets | int | starts 0, `+= 1` only when a new fleet forms | the answer |
| `cur`    | arrival time of slowest fleet ahead | float | starts 0, set to `t` only on new-fleet branch | the reference a car compares against to decide merge vs new |
| `p`      | a popped car's position | int | reassigned each pop | unpacked from the tuple but unused in the decision |

---

# Visual Walkthrough

Input: `target = 10, position = [4,1,0,7], speed = [2,2,1,1]`

Solo times:

```
pos 4: (10-4)/2 = 3.0
pos 1: (10-1)/2 = 4.5
pos 0: (10-0)/1 = 10.0
pos 7: (10-7)/1 = 3.0
```

Road layout (front = closest to target on the right):

```
mile:  0      1        4       7       10 (target)
car:  C0     C1       C4      C7        FINISH
 t:  10.0   4.5      3.0     3.0
      └──── behind ──────── ahead ────┘
```

Sorted by position ascending: `[(0,10.0), (1,4.5), (4,3.0), (7,3.0)]`. We pop from the end.

| Step | Popped (pos, t) | cur before | t > cur? | Action | fleet | cur after |
|------|-----------------|------------|----------|--------|-------|-----------|
| 1 | (7, 3.0)  | 0    | 3.0 > 0    → yes | new fleet | 1 | 3.0  |
| 2 | (4, 3.0)  | 3.0  | 3.0 > 3.0  → no  | merge     | 1 | 3.0  |
| 3 | (1, 4.5)  | 3.0  | 4.5 > 3.0  → yes | new fleet | 2 | 4.5  |
| 4 | (0, 10.0) | 4.5  | 10.0 > 4.5 → yes | new fleet | 3 | 10.0 |

Returns `3`. ✓

Example 1 check: `target=10, position=[1,4], speed=[3,2]` → times both `3.0`, sorted `[(1,3.0),(4,3.0)]`. Pop (4,3.0): `3>0` → fleet 1, cur 3. Pop (1,3.0): `3>3` → no, merge. Returns `1`. ✓

---

# Optimization Journey

1. **Brute Force:** simulate positions over time, or compare every pair of cars to see who catches whom — O(n²) and fiddly.
2. **Bottleneck:** tracking actual positions over time is unnecessary detail; pairwise catch-up checks double-count chained fleets.
3. **Observation:** I only need each car's *time to target*, and a car is only ever affected by the single slowest fleet directly ahead — not the whole history.
4. **Optimization:** sort by position, scan front-to-back, carry one scalar (`cur`) = slowest leader ahead.
5. **Final Approach:** O(n log n) from the sort, O(1) extra tracking beyond the stored pairs. An interviewer reaches this by asking "what's the only thing a trailing car cares about?" → the slowest thing in front → one running value.

---

# Pattern Recognition

## Signal Phrases

- "can not pass another car ahead of it" — order-dependent, front car constrains back cars
- "catch up to another car and then drive at the same speed" — absorbing / merging behavior
- "arrive at the destination" — convert to a single per-element quantity (time)

## Pattern Used

Monotonic stack / greedy single-pass with a running max.

## Why This Pattern Fits

- Cars get "absorbed" into the fleet ahead — classic push/keep-or-collapse behavior.
- Only the current slowest leader matters, so the stack collapses to one scalar (`cur`).
- Processing in resolved order (front-to-back) means each decision is final.

---

# Pattern Comparison

| Pattern | Why It Fits | Why It Doesn't | Verdict |
|---------|-------------|----------------|---------|
| Monotonic Stack | absorbing behavior, only-slowest-ahead-matters | don't strictly need to store the stack | core pattern (collapsed to a scalar here) |
| Two Pointers | it's the current NeetCode section | cars don't converge from both ends; no meet-in-middle | red herring — section bias |
| Sorting | needed to put cars in road order | it's setup, not the algorithm | prerequisite, not the pattern |
| Set (unique times) | times equal → looks like a fleet | equal times ≠ fleet; different times can share a fleet | wrong |

---

# Common Mistakes

- Writing the physics as "position at time t" instead of solving for time-to-target.
- Counting unique times in a set (coincidentally passes example 2, wrong in general).
- Comparing adjacent cars pairwise instead of against the slowest fleet ahead — breaks on chained merges.
- Initializing the reference (`cur`) to the front car's time, robbing the first fleet of its count.
- Updating the reference on the merge branch instead of only on the new-fleet branch.

---

# Edge Cases

| Case | Input | Expected | How Handled |
|------|-------|----------|-------------|
| single car | one element | 1 | first pop: `t > 0` → fleet 1 |
| equal times, same fleet | two cars same `t`, back one behind | merge | `t <= cur` (e.g. `3.0 > 3.0` is False) → no new fleet |
| faster car behind slower | back car smaller t | merge | `t <= cur` → no increment |
| slower car behind faster | back car bigger t | separate | `t > cur` → new fleet, cur updates |

---

# Debugging Lessons

- Hand-tracing example 2 and watching it return `2` instead of `3` pinpointed the init bug every time — the front car was comparing against itself.
- The fix for "first element never counts" is initializing the reference *below every possible value* (`cur = 0`, since all times are positive).
- `cur` must only update when a new fleet forms; updating it on merges makes the reference too fast and wrongly splits later cars.

---

# NeetCode / Official Solution

(Provided by me as a screenshot during the chat.)

## Code

```python
class Solution:
    def carFleet(self, target: int, position: List[int], speed: List[int]) -> int:
        pair = [[p, s] for p, s in zip(position, speed)]

        stack = []
        for p, s in sorted(pair)[::-1]:           # reverse sorted order
            stack.append((target - p) / s)
            if len(stack) >= 2 and stack[-1] <= stack[-2]:
                stack.pop()
        return len(stack)
```

## Comparison to My Solution

| | Mine (`cur` scalar) | NeetCode (stack) |
|---|---|---|
| sort | ascending by position, pop from end | `sorted(pair)[::-1]`, iterate front-to-back |
| structure | one running scalar `cur` | keeps a real stack of surviving fleet times |
| count | add 1 when `t > cur` (new fleet) | `len(stack)` of survivors after popping merges |
| pop rule | n/a | pop when `stack[-1] <= stack[-2]` (back catches front) |
| extra space | O(1) tracker | O(n) stack |

Same algorithm, mirrored bookkeeping: NeetCode counts survivors by *removing* merges; I count fleets by *adding* non-merges. My `cur` is exactly NeetCode's `stack[-1]` after popping settles — since only the slowest leader ahead ever matters, the stack collapses to one variable. Mine uses O(1) extra tracking; if asked "can you reduce space," the answer is "the stack isn't needed, one scalar suffices."

---

# Deep Understanding

The non-obvious part is that **time-to-target is a complete proxy for the entire trajectory.** You never simulate motion. If car B (behind) has a smaller solo time than car A (ahead), B *must* collide with A's rear before the line — there's no way for a faster car starting behind to reach the destination first without passing, which is forbidden. So a single time comparison replaces a continuous physics simulation. The second subtle bit: `cur = 0` works as a sentinel only because all times are strictly positive (`speed > 0`, `position < target`), guaranteeing the first car always trips `t > cur`.

---

# Complexity Analysis

| Metric | Complexity | Reason |
|--------|------------|--------|
| Time   | O(n log n) | the sort dominates; build and scan are each O(n) |
| Space  | O(n)       | the list of n `(position, t)` pairs; the `cur` tracker is O(1) |

---

# Related Problems

| # | Problem | Difficulty | Same Pattern? | Notes |
|---|---------|------------|---------------|-------|
| 739 | Daily Temperatures | Medium | Monotonic stack | classic monotonic stack, kept explicitly |
| 84 | Largest Rectangle in Histogram | Hard | Monotonic stack | stack of indices, collapse on smaller bar |
| 901 | Online Stock Span | Medium | Monotonic stack | running comparison against prior values |

---

# Interview Explanation (30 Seconds)

Convert each car to its solo time-to-target, sort by position, and scan from the car closest to the destination backward. Keep the slowest arrival time seen so far; any car slower than that starts a new fleet, anything faster merges. O(n log n) for the sort.

---

# Interview Explanation (2 Minutes)

The signal is "can't pass the car ahead" plus "catch up and match speed" — absorbing behavior, which points at a monotonic stack. The key reduction: I don't need positions over time, just each car's solo time to reach the target, `(target - position) / speed`. A car behind catches a car ahead exactly when its solo time is `<=` the ahead car's time (it'd arrive sooner unobstructed, so it must hit the back). I sort by position and process front-to-back so the fleet ahead is always resolved first. I carry one value, `cur`, the slowest arrival time ahead. If the current car's time exceeds `cur`, nothing ahead can stop it — new fleet, and it becomes the new bottleneck. Otherwise it merges, no count. Init `cur` to 0 so the very first car always counts. Complexity is O(n log n) time from the sort, O(n) space for the pairs. Tradeoff worth mentioning: the explicit monotonic stack and the single-scalar version are the same algorithm — the scalar is just the stack collapsed, since only the top (slowest leader) ever matters.

---

# Common Follow-Up Questions

- **"Can you avoid floats?"** Yes — compare `(target - p_back) * s_ahead` vs `(target - p_ahead) * s_back` via cross-multiplication to dodge division precision. For these constraints floats are fine.
- **"Why sort by position and not speed?"** Catch-up depends on who's physically ahead on the road, which is position order. Speed alone doesn't tell you the road layout.
- **"Can you reduce space?"** Drop the stack — only the slowest leader ahead matters, so one scalar (`cur`) replaces it.
- **"What if a car starts at the target?"** Constraint says `position < target`, so it can't, but it'd just be a zero-time fleet.

---

# Key Takeaways

- Reduce each element to the single quantity the problem actually depends on (time-to-target), not raw position/speed.
- Process in already-resolved order (front-to-back) so each decision is final.
- A monotonic stack often collapses to one running value when only the extreme element matters.

---

# Revision Notes

In 1 week / 1 month, refresh these three things only: (1) the formula `t = (target - position) / speed`, (2) sort by position, scan from biggest position backward, (3) carry `cur` starting at 0, increment fleet on `t > cur` and update `cur` there only. The two failure points to re-internalize are the `cur = 0` init and the `cur = t` placement *inside* the if.

---

# Final Mental Model

Think of cars as a single-file line at a toll road that can't be broken — nobody overtakes. The only thing a driver behind cares about is how slow the slowest car ahead of them is, because that's the speed they'll be stuck at the moment they catch up. So walk the line from the front, remembering the slowest leader so far. Anyone who would've finished even slower than that leader can't catch them — they start a fresh, even-slower clump that everyone further back now has to reckon with. Count the clumps.
