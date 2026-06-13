# 121. Best Time to Buy and Sell Stock

## Problem

You're given an array `prices` where `prices[i]` is the price of a stock on day `i`. You may buy on one day and sell on a **later** day. Return the maximum profit you can make, or `0` if no profitable trade exists.

**Example:** `prices = [7,1,5,3,6,4]` → buy at `1` (day 1), sell at `6` (day 4) → profit `5`.

---

# My Learning Journey

Started this one in the wrong shape entirely — reached for a converging two-pointer move (the kind that worked on 11 and 167), driving an inner pointer to hunt the lowest future price. Wrong instinct here: buy has to come before sell, so the right pointer can never walk backward past the left. That's the red herring for this problem.

The conceptual gap that actually cost me was **subtraction direction**. I knew in plain English I wanted buy-low / sell-high — I said it myself — but I wrote the subtraction backwards twice (`prices[l] - sell`, then `minimum - prices[r]`), which made the code reward losing trades. Once I started saying "sell minus buy" in my head in that exact order, it stuck.

After that the only thing standing between me and green was Python mechanics: I had three different spellings of `minimum` floating around (`mimimum` / `minimum` / `mimimum`) and they referred to different variables, which threw `NameError` and wrong answers. It only passed once I made all the spellings *consistent* — even though they were consistently wrong.

The real growth moment: after the first accepted version (which beat only ~15% on a noisy runtime metric), I stripped the solution down myself — dropped the unused `l` and `currlow`, folded the min-update into one line, started the range at 1 — without being told which lines to cut. That leaner version beat 100%. More importantly it removed the extra variables that were the *source* of the spelling bug in the first place.

---

# Attempt 1

```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        profit = 0
        for l in range(len(prices)):
            r = l + 1
            sell = prices[l]
            while r < len(prices) and prices[r] < sell:
                sell = prices[r]
                r += 1
            cur = prices[l] - sell
            profit = max(cur, profit)
        return profit
```

**What I was thinking:** treat `l` as the buy day, walk `r` forward looking for a sell price.

**What went wrong:**
- **Direction inverted** — `cur = prices[l] - sell` is buy minus sell.
- **Hunting the wrong extreme** — the inner `while prices[r] < sell` drives `sell` *down* to the lowest future price, and `break`s the instant a price ticks back up, so it never sees the real peak.
- Net effect: returned `0` on `[1,2,3,4,5]` (should be 4) and `4` on `[5,4,3,2,1]` (should be 0). It rewarded crashes.

---

# Attempt 2

```python
mimimum = prices[l]
for r in range(len(prices)):
    while l < r:
        currlow = prices[l]
        l += 1
        minimum = min(currlow, minimum)
    curr = minimum - prices[r]
    profit = max(profit, curr)
```

**What went wrong:**
- **Typo bug** — initialized `mimimum` but line below read `minimum` → `NameError` on first pass.
- **Direction still inverted** — `curr = minimum - prices[r]`, same backwards subtraction as Attempt 1, different variables.

---

# Attempt 3

```python
mimimum = prices[l]        # line 5
...
minimum = min(minimum, currlow)   # line 9
curr = prices[r] - mimimum        # line 10  -- direction fixed
```

**What went wrong:** direction was finally correct (`prices[r] - mimimum`), but now **three different spellings** existed — `mimimum` (5), `minimum` (9), `mimimum` (10). Line 9 updated one variable, line 10 read another that never changed from `prices[0]`. Failed 0/32.

---

# Accepted Solution(s)

### First accept (worked, but overbuilt — beat ~15% on noisy runtime)

```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        profit = 0
        l = 0
        mimimum = prices[l]
        for r in range(len(prices)):
            currlow = prices[l]
            l += 1
            mimimum = min(mimimum, currlow)
            curr = prices[r] - mimimum
            profit = max(profit, curr)
        return profit
```

Passed only because all spellings of `mimimum` were now *consistent*. `l` marches in lockstep with `r`, so `currlow`/`l`/the increment are all redundant.

### Final clean version (beat 100% runtime — implicit left edge)

```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        profit = 0
        l = 0
        mimimum = prices[l]
        for r in range(1, len(prices)):
            mimimum = min(mimimum, prices[r])   # left edge slides up
            curr = prices[r] - mimimum
            profit = max(profit, curr)
        return profit
```

> **Note to future me:** rename `mimimum` → `minimum` everywhere in the repo. It works only because it's consistent. Don't copy the typo into the next problem.

---

# Why It Works

**Invariant:** at every day `r`, `mimimum` holds the cheapest price seen on day `r` or earlier. So `prices[r] - mimimum` is the best possible profit *if you sell on day r*. Taking the max of that over all days gives the global best — because the optimal trade sells on *some* day, and on that day this quantity equals the optimal profit.

**Key operation:** update the running min *before* computing today's profit, so on a new-low day the profit comes out `0`, never negative.

---

# Traversal Logic

How we move through the data:

- One pointer `r` walks left → right, one pass, each day visited exactly once.
- The window's **left edge** (cheapest buy-in) is tracked *implicitly* by the value in `mimimum` — no stored index, because the problem only needs the price, not the day.

```
prices: [ 7   1   5   3   6   4 ]
r ->      ^                          r=0 (or 1 in clean version)
              ^                      r advances...
                  ^
left edge (mimimum) only "moves" when a cheaper price appears
```

---

# Business Logic

The decision made at each step:

1. **Update buy-in:** `mimimum = min(mimimum, prices[r])` — is today cheaper than anything before? If so, that's the new cheapest place to have bought.
2. **Evaluate sell:** `curr = prices[r] - mimimum`; `profit = max(profit, curr)` — if I sold today, is that my new best?

Order matters: step 1 before step 2.

---

# Variable Roles

| Variable | Represents | Stores | How It Changes | Why It Exists |
|----------|------------|--------|----------------|---------------|
| `mimimum` | cheapest buy-in seen so far (window left edge) | int | `min()` with each new price | so any sell day can compute profit against the best prior buy |
| `profit` | best profit found so far | int | `max()` each iteration | the running answer |
| `curr` | profit if selling today | int | recomputed each pass | scratch value feeding the max |
| `r` | current sell day | int (index) | `+1` each loop (the `for`) | drives the single pass |

---

# Visual Walkthrough

Input `[7,1,5,3,6,4]`:

| Step (r) | prices[r] | mimimum before | Action | mimimum after | curr = prices[r]-min | profit |
|----------|-----------|----------------|--------|---------------|----------------------|--------|
| 0 | 7 | 7 | init | 7 | 0 | 0 |
| 1 | 1 | 7 | new low | 1 | 0 | 0 |
| 2 | 5 | 1 | — | 1 | 4 | 4 |
| 3 | 3 | 1 | — | 1 | 2 | 4 |
| 4 | 6 | 1 | — | 1 | 5 | **5** |
| 5 | 4 | 1 | — | 1 | 3 | 5 |

Returns **5**. ✓

---

# Optimization Journey

1. **Brute Force:** for every buy day, scan all later days for the best sell → O(n²).
2. **Bottleneck:** the inner scan re-examines future prices for every buy day.
3. **Observation:** I don't need to re-scan. As I move forward, the only thing that matters about the past is the single cheapest price seen so far.
4. **Optimization:** carry that minimum as a running value; compute "sell today" profit in O(1) per day.
5. **Final Approach:** one pass, two tracked numbers (min-so-far, best-profit), no inner loop → O(n).

---

# Pattern Recognition

## Signal Phrases

- "buy on one day, sell on a **later** day" → directional, one pass, left-before-right
- "maximum profit" → running max over a per-position quantity

## Pattern Used

Sliding Window (implicit left edge / running-min variant).

## Why This Pattern Fits

- A window `[buy day, sell day]` where the left edge only ever moves forward.
- The left boundary is summarized by a single running value (the min), not an index.
- One linear pass maintains the answer incrementally.

---

# Pattern Comparison

| Pattern | Why It Fits | Why It Doesn't | Verdict |
|---------|-------------|----------------|---------|
| Sliding Window | window = [buy, sell], left edge slides forward only | — | ✅ correct |
| Greedy | hold lowest price, ask "sell today?" each step | — | ✅ same thing, different name |
| Converging Two Pointers | l/r as buy/sell | r can never walk left past l; buy must precede sell | ❌ red herring |
| DP | "min so far" is a running state | overkill, no subproblem table needed | ⚠️ valid but heavy |

---

# Common Mistakes

- Subtracting `buy - sell` instead of `sell - buy` (inverts the sign — rewards losing trades).
- Hunting the lowest *future* price instead of the lowest *prior* price.
- Updating the min *after* computing profit (can yield a negative on a new-low day).

---

# Edge Cases

| Case | Input | Expected | How Handled |
|------|-------|----------|-------------|
| Only ascending | `[1,2,3,4,5]` | 4 | min stays 1, profit grows to 4 |
| Only descending | `[5,4,3,2,1]` | 0 | curr always ≤ 0, profit stays 0 |
| Single day | `[5]` | 0 | loop body never improves on 0 |
| Two equal | `[3,3]` | 0 | curr = 0 |

---

# Debugging Lessons

- **Consistency beats correctness for variable names** — three spellings of one variable cost two failed submissions. One name = nothing to mistype against.
- **Trace before submitting.** Dry-running `[2,1,4]` (expect 3) exposes the direction bug in ~20 seconds. Both inversions would've been caught pre-submit.
- **Fewer variables = smaller bug surface.** The typo bug only existed because `l` + `currlow` created extra names. Collapsing the structure removed the bug class entirely.

---

# NeetCode / Official Solution

(Included because I pasted it in chat.)

## Code

```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        l, r = 0, 1
        maxP = 0
        while r < len(prices):
            if prices[l] < prices[r]:
                profit = prices[r] - prices[l]
                maxP = max(maxP, profit)
            else:
                l = r
            r += 1
        return maxP
```

## Comparison to My Solution

| Aspect | Mine (running-min) | NeetCode (explicit pointers) |
|--------|--------------------|------------------------------|
| Left edge | implicit value in `mimimum` | explicit index `l` |
| Left update | `min(mimimum, prices[r])` every pass | `l = r` only on `prices[l] >= prices[r]` |
| Complexity | O(n) / O(1) | O(n) / O(1) |
| Readability | tighter (one tracked value) | clearer intent (visible buy/sell pointers) |

Both correct, same complexity. Mine summarizes the left boundary as a value; theirs keeps an actual index. Worth knowing the *equal* case (`prices[l] == prices[r]`) falls into NeetCode's `else` and resets `l = r` — harmless, since equal prices mean zero profit.

---

# Deep Understanding

The non-obvious move is that you never need to remember *which* day you'd buy — only the cheapest price so far. Collapsing the left pointer from an index into a single running value is what makes this look "not like sliding window" while still being exactly that. The window is real; its left wall is just stored as a number instead of a pointer.

---

# Complexity Analysis

| Metric | Complexity | Reason |
|--------|------------|--------|
| Time | O(n) | single pass, O(1) work per day |
| Space | O(1) | two/three scalars, no extra structure |

---

# Related Problems

| # | Problem | Difficulty | Same Pattern? | Notes |
|---|---------|------------|---------------|-------|
| 122 | Best Time to Buy/Sell Stock II | Medium | variant | multiple transactions allowed → greedy sum of all up-moves |
| 3 | Longest Substring Without Repeating Chars | Medium | yes (variable window) | the real `while`-shrink-from-left shape |
| 53 | Maximum Subarray | Medium | cousin (Kadane) | running aggregate + running max, same skeleton |

---

# Interview Explanation (30 Seconds)

It's a sliding window. Walk the prices once, carry the cheapest price seen so far, and at each day check what profit selling today would give. Track the max. One pass, O(n) time, O(1) space.

---

# Interview Explanation (2 Minutes)

Recognition signal: buy must come before sell, and we want a max over a per-day quantity — that's a forward-only window. The invariant is that `minimum` always holds the lowest price on or before the current day, so `prices[r] - minimum` is the best profit achievable selling on day `r`. I update the minimum first, then compute today's candidate profit, then take the running max — updating the min first guarantees a new-low day yields 0 rather than a negative. The window's left edge is represented by a value, not an index, because the problem only needs the price. Brute force is O(n²) by scanning all sell days for each buy day; the insight that the only relevant fact about the past is the single cheapest price collapses it to O(n). One tradeoff: this returns only the max profit, not the buy/sell days — if those were needed I'd track the index where the min was set.

---

# Common Follow-Up Questions

- **What if multiple transactions are allowed?** → Problem 122: greedily sum every positive day-to-day delta.
- **Return the buy and sell days, not just profit?** → store the index when `minimum` updates, and the index of `r` when `profit` improves.
- **What if you can short (sell high, buy low later)?** → symmetric: also track a running max and the best `max - later_price`.
- **Cooldown / transaction fee variants?** → moves into DP (309, 714).

---

# Key Takeaways

- Sliding window doesn't require a visible second pointer — a running min/max/sum *is* the moving edge.
- Say the subtraction out loud as "sell minus buy" to kill the direction bug.
- One variable name, spelled once, beats three spellings — collapse structure to shrink the bug surface.

---

# Revision Notes

**1 week:** re-derive why updating the min *before* computing profit matters (new-low day → 0, not negative). Re-trace `[7,1,5,3,6,4]` → 5.

**1 month:** just recall the one-liner — carry the cheapest price, ask "sell today?" each day, take the max. That's the whole problem.

---

# Final Mental Model

You're walking down a row of daily prices with a sticky note in your pocket that always shows the cheapest price you've passed. At every new price you ask one question: "if I sold right now against my sticky note, how much would I make?" You remember the best answer you ever got, and you update the sticky note whenever you pass something cheaper. That sticky note is the left edge of your window — you never need to remember *where* you saw the cheap price, only *that* it was cheap.
