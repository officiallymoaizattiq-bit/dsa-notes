# 739 — Daily Temperatures

**Pattern:** Monotonic Stack
**Difficulty:** Medium
**NeetCode section:** Stack
**Time:** O(n) | **Space:** O(n)

---

# Problem

Given a list of daily temperatures, return a list where each element answers: how many days do you have to wait until a warmer temperature? If no warmer day comes after that day, the answer is 0.

### Example

```text
Input:  [73, 74, 75, 71, 69, 72, 76, 73]
Output: [ 1,  1,  4,  2,  1,  1,  0,  0]
```

Day 0 (73°) → warmer day is day 1 (74°) → wait = 1.
Day 2 (75°) → warmer day is day 6 (76°) → wait = 4.
Day 6 (76°) → no warmer day after it → wait = 0.

---

# Pattern Recognition

## Recognition Signals

- "Find the next greater element" framing.
- Asks how far until something larger / smaller appears.
- Output depends on a future element, not a past one.
- Single pass through the array should be possible.

## Pattern Used

Monotonic Decreasing Stack.

## Why This Pattern Fits

- Each day is "waiting" for a future warmer day. A stack naturally holds the waiting items.
- When a warmer day arrives, it resolves all previous days colder than itself in one sweep.
- Each element is pushed once and popped at most once → amortized O(n).

---

# Brute Force First

```python
def dailyTemperatures(self, temperatures):
    res = [0] * len(temperatures)
    for i in range(len(temperatures)):
        for j in range(i + 1, len(temperatures)):
            if temperatures[j] > temperatures[i]:
                res[i] = j - i
                break
    return res
```

**Why it's slow:** O(n²) — for every day, you re-scan the rest of the array looking for a warmer day. Tons of redundant work because you re-walk regions you've already seen.

**The question that leads to the optimal:** Instead of each day searching forward for its warmer day, what if days *waited* and the warmer day *found them*?

---

# Solutions

## Solution 1 — My Accepted Solution

### Status

[Fill in after solving cold]

### Full Code

```python
class Solution:
    def dailyTemperatures(self, temperatures: List[int]) -> List[int]:
        res = [0] * len(temperatures)
        stack = []  # pair: [temp, index]

        for i, t in enumerate(temperatures):
            while stack and t > stack[-1][0]:
                stackT, stackInd = stack.pop()
                res[stackInd] = (i - stackInd)
            stack.append([t, i])

        return res
```

### Core Idea

- Walk through temperatures once.
- Stack holds days still waiting for a warmer day, as `[temp, index]` pairs.
- When current day is warmer than the stack top, that top day just found its warmer day. Pop it, record the distance.
- Repeat the popping while the current day keeps resolving stack entries.
- Then push the current day onto the stack — it now waits.

### Time Complexity

O(n) — each element pushed once, popped at most once.

### Space Complexity

O(n) — stack worst case holds the entire array (strictly decreasing input).

### Tradeoffs

Pros:
- Single pass.
- Optimal complexity.
- Each step has a clear meaning.

Cons:
- Easy to get the operand order wrong on first try.
- Requires recognizing the monotonic stack pattern up front.

---

## Solution 2 — NeetCode Style Solution

### Status

Accepted

### Full Code

```python
class Solution:
    def dailyTemperatures(self, temperatures: List[int]) -> List[int]:
        res = [0] * len(temperatures)
        stack = []  # pair: [temp, index]

        for i, t in enumerate(temperatures):
            while stack and t > stack[-1][0]:
                stackT, stackInd = stack.pop()
                res[stackInd] = (i - stackInd)
            stack.append([t, i])

        return res
```

### Core Idea

Identical to Solution 1. NeetCode's canonical version IS this. There isn't a meaningfully more compact form — the structure is already minimal.

### Time Complexity

O(n)

### Space Complexity

O(n)

### Tradeoffs

Pros:
- Idiomatic.

Cons:
- None — this is the optimal expression of the pattern.

---

# Variable Roles

| Variable | What it Stores | What it Represents |
|----------|----------------|---------------------|
| `res`    | Array of ints  | Final answer; day `i` waited `res[i]` days for a warmer day |
| `stack`  | List of `[temp, index]` | Days that haven't found their warmer day yet |
| `i`      | Int            | Current index in the temperatures array |
| `t`      | Int            | Current temperature being processed |
| `stackT` | Int            | Temperature of the popped day |
| `stackInd` | Int          | Original index of the popped day |

---

# Traversal Logic

How data moves through the algorithm.

```text
Read (i, t) from temperatures
↓
Is stack non-empty AND t > top temp on stack?
↓
Yes → pop top, compute res[stackInd] = i - stackInd, repeat
↓
No  → push [t, i] onto stack
↓
Move to next (i, t)
```

---

# Business Logic

### Invariant

```text
At the start of each iteration, the stack contains days whose warmer
day has not yet been found, in strictly decreasing temperature order
from bottom to top.
```

### Decision Rule

```text
Current temp > stack top temp → top day just got resolved → pop and record
Current temp <= stack top temp → current day starts waiting → push
```

The "while" loop (not "if") matters: a single hot day can resolve multiple stacked cold days at once.

---

# Visual Trace

Input:

```text
[73, 74, 75, 71, 69, 72, 76, 73]
```

| i | t  | Stack Before | Action | Stack After | res |
|---|----|--------------|--------|-------------|-----|
| 0 | 73 | `[]` | push | `[[73,0]]` | `[0,0,0,0,0,0,0,0]` |
| 1 | 74 | `[[73,0]]` | 74>73, pop [73,0], res[0]=1, push | `[[74,1]]` | `[1,0,0,0,0,0,0,0]` |
| 2 | 75 | `[[74,1]]` | 75>74, pop [74,1], res[1]=1, push | `[[75,2]]` | `[1,1,0,0,0,0,0,0]` |
| 3 | 71 | `[[75,2]]` | 71<75, push | `[[75,2],[71,3]]` | same |
| 4 | 69 | `[[75,2],[71,3]]` | 69<71, push | `[[75,2],[71,3],[69,4]]` | same |
| 5 | 72 | `[[75,2],[71,3],[69,4]]` | 72>69 pop res[4]=1, 72>71 pop res[3]=2, 72<75 push | `[[75,2],[72,5]]` | `[1,1,0,2,1,0,0,0]` |
| 6 | 76 | `[[75,2],[72,5]]` | 76>72 pop res[5]=1, 76>75 pop res[2]=4, push | `[[76,6]]` | `[1,1,4,2,1,1,0,0]` |
| 7 | 73 | `[[76,6]]` | 73<76, push | `[[76,6],[73,7]]` | `[1,1,4,2,1,1,0,0]` |

Result:

```text
[1, 1, 4, 2, 1, 1, 0, 0]
```

Stack still has `[76,6]` and `[73,7]` at the end. They never found a warmer day, so their `res` entries stay 0. Correct.

---

# Edge Cases

## All Decreasing Input

```text
Input: [80, 75, 70, 65]
Expected: [0, 0, 0, 0]
```

Nothing ever pops — the stack grows to the full length. `res` stays all zeros.

## All Increasing Input

```text
Input: [70, 75, 80, 85]
Expected: [1, 1, 1, 0]
```

Every day except the last gets popped immediately by the next day. Each `res` entry is 1. Last day never pops, stays 0.

## All Same Temperature

```text
Input: [75, 75, 75, 75]
Expected: [0, 0, 0, 0]
```

`t > stack[-1][0]` uses strict `>`, not `>=`. Equal temps don't resolve each other. All stay 0.

## Single Element

```text
Input: [73]
Expected: [0]
```

Loop runs once, nothing on the stack to pop, single element pushed and never resolved.

---

# Bugs I Actually Made

[Fill in your own as they happen. Common ones on this problem:]

## Bug 1

What I wrote: `if stack and t > stack[-1][0]`
What I needed: `while stack and t > stack[-1][0]`
Type: Logic Bug
Why I missed it: I didn't realize a single hot day can resolve multiple stacked cold days in one step. Using `if` only resolves one day per iteration → wrong answer when multiple resolutions needed.

## Bug 2

What I wrote: `t >= stack[-1][0]`
What I needed: `t > stack[-1][0]`
Type: Off-by-one Bug
Why I missed it: I assumed "warmer or equal" counts, but the problem says strictly warmer. Equal days should keep waiting.

## Bug 3

What I wrote: `res[stackInd] = stackInd - i`
What I needed: `res[stackInd] = i - stackInd`
Type: Wrong Operand Order
Why I missed it: I thought of "from the popped day to today" but flipped the subtraction direction. Today is later, so today's index is larger.

## Bug 4

What I wrote: stored just temperatures on stack, no index
What I needed: stored `[temp, index]` pairs
Type: Logic Bug
Why I missed it: I forgot that I need to write to `res` at the popped day's index, not at the current `i`. Without storing the index, I have no way to know where the popped day came from.

---

# Self-Check Questions

Answer all of these without looking at the code:

1. Why a stack and not a queue? → LIFO; the most recent unresolved day is always the first to be resolved by the next warmer day.
2. Why is the stack monotonically *decreasing* from bottom to top? → Because any day that gets pushed is colder than (or equal to) what's already on top. Warmer days resolve and pop the smaller ones before pushing.
3. What's the time complexity and why? → O(n). Each element pushed once and popped at most once = at most 2n operations.
4. What's the worst-case space? → O(n) when input is strictly decreasing — every element gets pushed and nothing is popped until the loop ends.
5. Why `while` instead of `if`? → A single warm day can resolve multiple stacked cold days in one iteration.
6. Why store the index along with the temperature? → To know where to write the answer in `res` once the day gets resolved.
7. What invariant does the stack maintain? → It always holds unresolved days in strictly decreasing temperature order.

If any answer is shaky, re-read that section.

---

# Sibling Problems (Same Pattern)

| # | Problem | Difficulty | Notes |
|---|---------|------------|-------|
| 496 | Next Greater Element I | Easy | Same pattern, simpler framing. Bank this next. |
| 503 | Next Greater Element II | Medium | Circular array twist. Run the array twice. |
| 901 | Online Stock Span | Medium | Same pattern but as a streaming class. |
| 84 | Largest Rectangle in Histogram | Hard | Classic — monotonic stack with width tracking. |
| 42 | Trapping Rain Water | Hard | Solvable with monotonic stack OR two pointers. |

Do 496 next — pattern transfers directly.

---

# Interview Explanation (30 Seconds)

"This is a next-greater-element problem, which signals a monotonic stack. I keep a stack of days that haven't found a warmer day yet, storing each as a `[temp, index]` pair. As I walk the array, whenever the current day is warmer than the top of the stack, that top day just found its answer — I pop it and record `current_index - popped_index`. I keep popping while the current day keeps resolving stack entries, then push the current day. Each element is pushed once and popped once, giving O(n) time and O(n) space."

---

# Interview Explanation (2 Minutes)

Reverse the problem: instead of each day searching forward for its warmer day, days wait on a stack until a warmer day arrives and resolves them.

The pattern is a **monotonic decreasing stack** — the stack always holds unresolved days in strictly decreasing temperature order. The recognition signal is "find the next greater element" or "how far until something bigger."

Invariant:

```text
At the start of each iteration, the stack holds days whose warmer day
hasn't been found yet, in decreasing temperature order bottom-to-top.
```

Walking the array, for each `(i, t)`:
- While the current temperature beats the top of the stack, the top day just found its warmer day. Pop it and record `res[popped_index] = i - popped_index`.
- Then push `[t, i]` onto the stack — current day starts waiting.

The `while` loop matters: one warm day can resolve multiple stacked cold days. Storing both temp and index is required because the answer must be written at the popped day's index, not the current one. The comparison uses strict `>` because the problem demands strictly warmer.

Complexity:
- Time: O(n) — amortized; each element pushed once, popped at most once.
- Space: O(n) — worst case is strictly decreasing input, where nothing ever pops.

Tradeoff: simpler-looking O(n²) brute force exists, but times out on large inputs. The monotonic stack is the canonical optimization.

---

# Complexity Summary

| Metric | Complexity | Reason |
|--------|------------|--------|
| Time   | O(n)       | Each element pushed once, popped at most once → 2n ops max |
| Space  | O(n)       | Stack worst case holds the entire array |

---

# Mental Model

When the problem asks "what's the next thing bigger than me," don't have each element search forward. Have elements **wait in line** on a stack, and let the next big thing **walk in and resolve everyone smaller than itself** in one sweep. That's monotonic stack thinking. The stack is a waiting room. The new arrival kicks everyone out who was waiting for someone like them.

---

# How to Actually Lock This In

1. **Solve it cold tomorrow.** No notes, no editorial, 30-min timer.
2. **Solve it cold 3 days later.** Same protocol.
3. **Solve it cold 1 week later.** If it's automatic by now, you own it.
4. **Move to LC 496 (Next Greater Element I)** — the pattern is banked, leverage it.
