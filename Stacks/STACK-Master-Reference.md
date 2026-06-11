# Stack — Master Reference (NeetCode 150)

**Section complete:** 20, 155, 150, 739, 853, 84 *(22 Generate Parentheses is filed here on NeetCode but is a backtracking problem — see note at the very end).*

**How to use this doc:** skim Parts 1–5 every month (the templates and your bug patterns), glance at the Problem Cards in Part 6 only if a specific one feels foggy, and read Part 9 in 60 seconds the day before an interview. If you can reproduce the Monotonic Stack template in Part 3 from memory, you own this entire section.

---

# Part 1 — The Stack Decision Tree

The question that routes everything: **"Does the answer for the current element depend on the most-recently-seen unresolved element?"** If yes → stack. Stacks exist for exactly one reason: *the most recent thing matters most* (LIFO).

```
Problem arrives
   │
   ├─ "matching pairs / nested / close in reverse order"
   │        → PLAIN STACK (push opens, pop+check on close)         → 20 Valid Parens
   │
   ├─ "evaluate a postfix / build a value as you read"
   │        → STACK AS ACCUMULATOR (push operands, operator pops 2) → 150 RPN
   │
   ├─ "query something (min/max) about the stack in O(1)"
   │        → STACK + AUXILIARY STACK (mirror the answer)           → 155 Min Stack
   │
   ├─ "next greater / next smaller / how far until bigger"
   │        → MONOTONIC STACK (elements wait, the answer evicts them)→ 739 Daily Temps
   │
   ├─ "max rectangle / span bounded by nearest smaller on both sides"
   │        → MONOTONIC STACK + WIDTH (boundaries both sides)        → 84 Histogram
   │
   └─ "process in a forced order, later items absorb earlier ones"
            → SORT + STACK (sort first, then one monotonic pass)     → 853 Car Fleet
```

**The meta-signal:** if you find yourself wanting to "look back at the last unresolved thing," or you're writing an O(n²) loop that re-scans backward/forward for "the nearest X," a stack collapses it to O(n).

---

# Part 2 — The Five Stack Archetypes

| # | Archetype | What the stack holds | The core move | Section problem |
|---|-----------|----------------------|---------------|-----------------|
| 1 | **Matching / LIFO validation** | unmatched openers | push on open, pop+compare on close, must end empty | 20 |
| 2 | **Accumulator / evaluator** | fully-evaluated subexpressions | operator pops its operands, pushes the result | 150 |
| 3 | **Stack + auxiliary** | values + a parallel "answer" stack | every push/pop updates both stacks in lockstep | 155 |
| 4 | **Monotonic** | unresolved elements in sorted order | incoming element evicts everyone it "beats," recording their answer | 739, 84 |
| 5 | **Sort + stack** | survivors after an ordering | sort to force the order, then one monotonic-style pass | 853 |

Everything in this section is one of these five. Identify the archetype first; the code follows from it.

---

# Part 3 — The Monotonic Stack Master Template (the crown jewel)

This single skeleton solves 739, 84, and the entire "next greater/smaller element" family (496, 503, 901, etc.). Memorize the skeleton + the decision table.

```python
stack = []                        # holds INDICES (or [value, index] pairs)
for i, x in enumerate(arr):
    while stack and <BREAKS_MONOTONIC(arr[stack[-1]], x)>:
        popped = stack.pop()      # this element just found its boundary
        # --- record popped's answer here, using i as its far boundary ---
    stack.append(i)               # x now waits; ALWAYS push, unconditionally, last
# optional: drain whatever's left if leftovers still need an answer
```

## The one rule that decides everything

> **Keep the stack monotonic in the OPPOSITE direction of what you're hunting.**
> Hunting the *next greater* → keep a **decreasing** stack. Hunting the *next smaller* → keep an **increasing** stack. When an incoming element breaks the order, it **is** the answer for everyone it evicts.

## Decision table

| You want, for each element… | Stack order (bottom→top) | Pop condition (`while stack and …`) | On pop, the popped element's… |
|------------------------------|--------------------------|--------------------------------------|-------------------------------|
| **next greater** element | decreasing | `arr[stack[-1]] < x` | right-bigger = `x` (at index `i`) |
| **next greater-or-equal** | decreasing | `arr[stack[-1]] <= x` | (equals also resolve) |
| **next smaller** element | increasing | `arr[stack[-1]] > x` | right-smaller = `x` |
| **nearest smaller on BOTH sides** (width) | increasing | `arr[stack[-1]] > x` | left wall = new `stack[-1]`, right wall = `i` |

## The two boundaries you get for free on every pop

```
          stack[-1]            i
   ... [left boundary] popped [right boundary] ...
              │          │            │
       what's left      the      the element
       under it after   element  that triggered
       the pop          you pop  the pop (current x)
```

- **Right boundary** = the current element `i` (the one that broke the order).
- **Left boundary** = whatever is left on the stack after popping (`stack[-1]`), or "the start" if the stack is now empty.

## Two flavors of "record the answer"

**Distance / next-element flavor (739):** store `[value, index]`, answer is `i - popped_index`.
```python
while stack and x > stack[-1][0]:
    val, idx = stack.pop()
    res[idx] = i - idx        # current minus popped — the later index is bigger
stack.append([x, i])
```

**Width / both-boundaries flavor (84):** store indices, width is the gap between the two walls.
```python
while stack and heights[stack[-1]] > heights[i]:
    popped = stack.pop()
    width = i if not stack else i - stack[-1] - 1   # -1 excludes the walls
    maxArea = max(maxArea, heights[popped] * width)
stack.append(i)
# drain leftovers with right wall = n:
while stack:
    popped = stack.pop()
    width = n if not stack else n - stack[-1] - 1
    maxArea = max(maxArea, heights[popped] * width)
```

## Why it's O(n), not O(n²) (say this in interviews)

The `while` inside the `for` looks quadratic but isn't. **Each index is pushed exactly once and popped at most once** → at most `2n` stack operations total. The inner loop's work is *amortized* O(1) per outer step. Naming "amortized" out loud signals you actually understand it.

## The drain / sentinel choice

Bars/elements still on the stack at the end never met their boundary. Two ways to handle:
- **Explicit drain:** a second `while stack:` loop with the right boundary set to `n`.
- **Sentinel:** append a `0` (for histogram) or `inf` (for next-smaller) to the input so the final element forces every leftover to pop inside the main loop. Cleaner, but less obvious to a reader.

---

# Part 4 — Python Stack Syntax Cheat Sheet

The literal lines you keep needing. (Several of these map directly to bugs you actually made — see Part 5.)

```python
stack = []                 # a list IS a stack in Python
stack.append(x)            # PUSH  — never stack.push(), that doesn't exist
top = stack[-1]            # PEEK  — does NOT remove
val = stack.pop()          # POP   — needs the (); stack.pop (no parens) = the method object
if stack: ...              # truthiness check = "is the stack non-empty"
if not stack: ...          # "is the stack empty"
for i, x in enumerate(a):  # index + value together — needed for monotonic answers

# truncate toward zero (RPN division) — floor division is WRONG for negatives:
int(a / b)                 # -10 / 6 -> -1   ✓
a // b                     # -10 // 6 -> -2   ✗

# inside a class, attributes need self.:
self.stack.append(x)       # not stack.append(x)

# pair storage when you need to remember where an element came from:
stack.append([value, i])   # then: val, idx = stack.pop()
```

Guard rule: **never read `stack[-1]` or call `stack.pop()` without first knowing the stack is non-empty** (`while stack and …` short-circuits left-to-right, so the `stack` check protects the index access).

---

# Part 5 — Your Recurring Bugs (the personalized part)

These are the mistakes that showed up across *multiple* problems in this section. They are your real failure modes — pre-check for them every time.

### Bug pattern 1 — Subtraction / operand order (your #1 recurring bug)

Showed up in **150** (`val2 -= val1`, division order), **739** (`i - stackInd`, not `stackInd - i`), **84** (`i - stack[-1] - 1`).

- **RPN:** second pop = left operand. `result = val2 OP val1`. (`10 3 -` → `10 - 3`.)
- **Distance:** current minus popped. The later element has the bigger index, so `i - popped_idx` is positive.
- **Cue:** "later minus earlier." Whichever thing happened *later in the array / on top later* is the minuend.

### Bug pattern 2 — `>` vs `>=` (equality handling)

Showed up in **739** (strict `>`), **84** (strict `>`, equals stay), **155** (`<=` to capture duplicate mins).

- Before writing the condition, ask: **what should equal elements do — wait or resolve?**
- Daily temps / histogram: equals should **keep waiting** → strict `>`.
- Min stack: duplicate mins must **both** be tracked or pop breaks → `<=` on push.

### Bug pattern 3 — `if` vs `while` on the pop

Showed up in **739**, **84**. One incoming element can resolve **many** stacked elements. Monotonic pops are **always `while`**, never `if`.

### Bug pattern 4 — Forgetting to store the index

Showed up in **739**, **84**. If you must write the answer back at the *original* position, store `[value, index]` (or store indices directly). Value-only stacks can't tell you where to write.

### Bug pattern 5 — Python literals

`stack.push` (→ `.append`), `stack.pop` without `()`, bare `stack` instead of `self.stack` inside a class, `//` instead of `int(a/b)`.

### Bug pattern 6 — Empty-stack access + push order

Reading `stack[-1]` / popping without an emptiness guard (**20**, **84**). And in monotonic problems: **pop first, then push, and push unconditionally** — a conditional push deletes elements on monotonic inputs (the `[4,3]→4-instead-of-6` bug in 84).

> **Pre-submit checklist (run this every time):**
> 1. Subtraction order — later minus earlier?
> 2. `>` vs `>=` — what do equals do?
> 3. `if` or `while` — can one arrival resolve many?
> 4. Did I store the index if I write back by position?
> 5. `.append` / `.pop()` / `self.` / `int(a/b)` correct?
> 6. Empty-stack guard before every `[-1]`/`pop()`? Push unconditional and last?

---

# Part 6 — Problem Cards

One compact card per problem. Signal → archetype → skeleton → the trick → your bug → complexity.

## 20 — Valid Parentheses

- **Signal:** matching pairs, nested, must close in reverse order.
- **Archetype:** plain stack (LIFO matching).
- **Skeleton:** push openers; on a closer, pop and check it matches the expected opener; invalid if stack empty on a closer or mismatched; **valid only if stack ends empty.**
- **Trick:** a `closeToOpen` hash map (`")":"(" , "]":"[" , "}":"{"`) collapses the comparison to one line: `if not stack or stack[-1] != closeToOpen[c]: return False`.
- **Why not a counter:** counts can't catch ordering — `([)]` has equal counts but is invalid.
- **Your bug:** `stack.pop` (no parens), missing empty-stack check, reversed final validation.
- **Complexity:** O(n) / O(n).

## 155 — Min Stack

- **Signal:** O(1) `getMin()` required; can't rescan.
- **Archetype:** stack + auxiliary stack.
- **Skeleton:** main stack for values; a `minStack` that holds the running minimum. On push, push `min(val, minStack[-1])`. On pop, pop **both**. `getMin` = `minStack[-1]`.
- **Trick:** the min stack is a *history* — when the current min is popped, the previous min is already sitting underneath, no recomputation.
- **Your bug:** a single min *variable* loses history (`push(5),push(2),pop()` should return 5); `<` instead of `<=` drops duplicate mins.
- **Complexity:** all ops O(1), space O(n).

## 150 — Evaluate Reverse Polish Notation

- **Signal:** postfix / RPN / "evaluate expression."
- **Archetype:** stack as accumulator.
- **Skeleton:** push numbers; on an operator, pop two (`r = pop()`, `l = pop()`), compute `l OP r`, push result. Return `stack[-1]`.
- **Trick:** invariant — the stack always holds fully-evaluated subexpression values. Division truncates toward zero → `int(l / r)`, not `//`.
- **Your bug:** `stack.push`, the always-true `s != "+" or s != "-" …` guard, flipped subtraction/division order, floor-div on negatives.
- **Complexity:** O(n) / O(n).

## 739 — Daily Temperatures

- **Signal:** "how many days until warmer" = next-greater-element.
- **Archetype:** monotonic **decreasing** stack.
- **Skeleton:** store `[temp, index]`; `while stack and t > stack[-1][0]:` pop and set `res[idx] = i - idx`; then push. Leftovers stay 0.
- **Trick:** reverse the problem — days *wait*, the warm day *finds* them. One warm day resolves many cold days (`while`).
- **Your bug:** `if` instead of `while`, `>=` instead of `>`, `stackInd - i` flipped, forgot to store the index.
- **Complexity:** O(n) amortized / O(n).

## 853 — Car Fleet

- **Signal:** items move in a forced order; later items get absorbed by slower ones ahead; count survivors.
- **Archetype:** **sort + stack.**
- **Skeleton:**
  ```python
  pair = sorted(zip(position, speed), reverse=True)   # closest to target first
  stack = []                                          # arrival times of fleet leaders
  for p, s in pair:
      time = (target - p) / s
      stack.append(time)
      if len(stack) >= 2 and stack[-1] <= stack[-2]:  # caught the fleet ahead
          stack.pop()                                 # absorbed — not a new fleet
  return len(stack)
  ```
- **Trick:** sort by position descending so you process cars front-to-back. Compute each car's *time to target*. A car merges into the fleet ahead iff its time ≤ the leader's time (it would catch up). The stack holds distinct fleet arrival times; `len(stack)` is the fleet count.
- **Watch:** the comparison is against the car directly **ahead** (top of stack), and `<=` means "arrives no later" = caught up.
- **Complexity:** O(n log n) (the sort dominates) / O(n).

## 84 — Largest Rectangle in Histogram

- **Signal:** max rectangle; each bar's reach bounded by the nearest *shorter* bar on both sides.
- **Archetype:** monotonic **increasing** stack + width.
- **Skeleton:** indices on the stack; `while stack and heights[stack[-1]] > heights[i]:` pop, `width = i if not stack else i - stack[-1] - 1`, update max with `heights[popped] * width`; push unconditionally; **drain** at the end with right wall = `n`.
- **Trick (the one that took longest):** width is the **gap between the two walls**, computed by index math — *never* a count of pops. A popped bar leaves a gap that earlier shorter bars stretch across; only `i - stack[-1] - 1` sees it. The `-1` excludes the walls themselves (fencepost). Empty stack → no left wall → `width = i`.
- **Your bug:** width as pop-count, `heights[i]` vs `heights[popped]`, `while stack:` draining the wall, area trapped in `else`, undefined `n`, push-before-pop deleting bars.
- **Complexity:** O(n) amortized / O(n).

---

# Part 7 — Mental Cues / Analogies (the memory hooks)

These are what you actually recall in 6 months, not the code.

- **Plain stack (20):** a stack of plates. Each opener adds a plate; each closer must remove the matching top plate. Wrong plate or no plate → invalid. Table clear at the end → valid.
- **Accumulator (150):** numbers wait on a conveyor; an operator grabs the two nearest, fuses them into one, drops it back. First grab = right operand.
- **Stack + aux (155):** the min stack is a stack of *receipts* for the minimum. When the current minimum leaves, the previous minimum's receipt is already underneath.
- **Monotonic (739/84):** a club with a strict bouncer. People line up shortest-to-tallest (or coldest-to-warmest). Someone waits because they don't yet know how far they reach. The instant someone "better" walks in, they evict everyone they beat — and the person still behind the evictee is automatically the other boundary. You measure the room between the two boundaries (the `-1` = "don't count the walls, just the room").
- **Sort + stack (853):** cars on a one-lane road to a finish line. You sort them by who's closest to the finish, then ask each one: do you catch the fleet ahead before the line? If yes, you disappear into it. Survivors = fleets.

**The universal monotonic mantra:** *"Elements wait in a line. The new arrival kicks out everyone it beats, and on the way out each one learns both its boundaries for free."*

---

# Part 8 — Traps Interviewers Set

1. **"Isn't that nested loop O(n²)?"** → No. Each element is pushed once and popped at most once; total inner-loop work is bounded by `n`. It's **amortized O(n)**.
2. **`>` vs `>=`** → they'll feed you equal elements (`[75,75,75]`, `[5,5]`) to see if your equality choice breaks. Know what equals should do.
3. **Operand order** → they'll use subtraction/division (non-commutative) specifically because flipping order is a silent wrong answer, not a crash.
4. **Write-back position** → "where does the answer go?" If it's the *popped* element's slot, you must have stored its index.
5. **Empty stack / boundaries** → first element, last element, strictly increasing, strictly decreasing. The drain and the empty-stack width branch are where these bite.
6. **Counter shortcut** → for matching problems, they'll offer `([)]` to kill any count-based approach and force the stack.

---

# Part 9 — 60-Second Monthly Refresh

Read only this for upkeep. If any line is fuzzy, jump to the matching Part above.

```
WHEN STACK: most-recent-unresolved-element matters → LIFO.

5 ARCHETYPES:
  matching pairs        → push open, pop+check close, end empty   (20)
  evaluate postfix      → operator pops 2 operands                (150)
  O(1) min/max query    → mirror it in an auxiliary stack         (155)
  next greater/smaller  → MONOTONIC stack                         (739)
  max span / rectangle  → MONOTONIC + width (i - stack[-1] - 1)   (84)
  absorbed-in-order     → SORT first, then stack                  (853)

MONOTONIC TEMPLATE:
  stack = []                       # indices or [val, i]
  for i, x in enumerate(arr):
      while stack and BREAKS_ORDER(top, x):
          popped = stack.pop()     # popped just found its boundary
          record(popped, i)        # right boundary = i; left = new stack[-1]
      stack.append(i)              # always, last, unconditional
  drain leftovers if needed (right boundary = n)

  hunting BIGGER  → DECREASING stack → pop when x >  top
  hunting SMALLER → INCREASING stack → pop when x <  top
  equals wait     → strict > or <    | it's O(n) amortized

PRE-SUBMIT CHECKLIST:
  1 subtraction = later − earlier
  2 > vs >= : what do equals do
  3 if vs while : one arrival can resolve many → while
  4 store index if writing back by position
  5 .append / .pop() / self. / int(a/b)
  6 empty-stack guard before [-1]; push last + unconditional
```

---

# Appendix — The One Outlier: 22 Generate Parentheses

NeetCode files **22 Generate Parentheses** under Stack, but it's really a **backtracking** problem — you build valid combinations by tracking open/close counts, not by pushing/popping a runtime stack. The only stack connection is conceptual (validity = balanced brackets). Treat it when you reach the Backtracking section; it does not belong to the templates above. No notes here because it wasn't solved in this section's work.

---

# Sibling Problems To Bank Next (pattern already transfers)

| # | Problem | Archetype | Why it's easy now |
|---|---------|-----------|-------------------|
| 496 | Next Greater Element I | monotonic decreasing | the baseline version of 739 |
| 503 | Next Greater Element II | monotonic decreasing | circular — loop the array twice |
| 901 | Online Stock Span | monotonic | same pattern as a streaming class |
| 85 | Maximal Rectangle | monotonic + width | literally calls 84 per matrix row |
| 42 | Trapping Rain Water | monotonic OR two-pointer | boundary thinking from 84 |

Do **496** first — it's 739 with the training wheels still on, and it confirms the template is yours.
