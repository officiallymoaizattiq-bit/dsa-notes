# 424. Longest Repeating Character Replacement

## Problem

Given a string `s` (uppercase English letters) and an integer `k`, you may replace up to `k` characters in the string with any uppercase letter. Return the length of the longest substring containing a single repeated character you can produce after at most `k` replacements.

**Example:** `s = "AABABBA"`, `k = 1` → `4` (e.g. window `"AABA"` → replace the one `B` → `"AAAA"`).

---

# My Learning Journey

First attempt was the wrong *model*, not just buggy code. I tried to track a running "swaps left" counter (`swapsleft = k`, decrement on mismatch) and compared `s[l] == s[r]` — a two-pointer "are these endpoints equal" move. That doesn't apply here: it would infinite-loop (`l` never moved inside the `while`) and the trailing `l = r` collapsed the window every iteration. The bugs were symptoms of anchoring on the wrong question.

The reframe was the key moment. The right question for any window is "how many chars would I have to replace to make this window uniform?" — and the cheapest answer is keep the most frequent char, replace the rest:

```
replacements needed = window_size - maxfreq
window is valid while that is <= k
```

I worked the model out myself through the concept questions: a **set isn't enough** here (I needed counts, not membership → dict / `defaultdict(int)`), the validity condition is `size - maxfreq <= k`, and the shrink fires when `size - maxfreq > k`. I also reasoned out that a count of 0 left in the dict is harmless, because `max(count.values())` ignores zeros (there's always a char with count ≥ 1 in a non-empty window).

The optimality insight I nailed last: **why max freq and not some other char?** The cost of targeting char `X` is `size - count(X)`; `size` is fixed, so minimizing cost means maximizing `count(X)` — keep the char you already have the most of, fewest left to replace. So `size - maxfreq` is provably the minimum replacement cost, and I can re-derive the formula instead of just recalling it.

**Honest about the solve:** the *concept* was clean and solo (no AI on the algorithm), but the *execution* was not a one-shot. I had multiple failed submissions fixing syntax, and I had to google the `freq.get(key, 0)` idiom because I didn't remember the exact syntax. So this is a **tries = 3 (mechanical)** solve, same bucket as 121 — the thinking was right, the Python vocabulary was the tax. Logged honestly.

---

# Attempt 1 (wrong model)

```python
class Solution:
    def characterReplacement(self, s: str, k: int) -> int:
        swapsleft = k
        longest = 0
        l = 0
        for r in range(len(s)):
            while s[l] == s[r] or swapsleft > 0:
                if s[l] != s[r]:
                    swapsleft -= 1
                longest = max(longest, r - l + 1)
            l = r
        return longest
```

**Why it can't work:**

| Issue | Type | Explanation |
|-------|------|-------------|
| `l` never moves inside the `while` | Logic / infinite loop | if `s[l] == s[r]` the condition stays true forever; nothing changes |
| compares `s[l] == s[r]` | Wrong model | this is a 2-pointer endpoint check; 424 is about window *frequencies*, not endpoint equality |
| `l = r` after the loop | Logic | collapses the window to a single index every iteration — destroys the thing being measured |
| `swapsleft` as a running counter | Wrong model | the budget isn't a decrementing counter; it's `size - maxfreq` recomputed per window |

---

# Accepted Solution

```python
from collections import defaultdict

class Solution:
    def characterReplacement(self, s: str, k: int) -> int:
        longest = 0
        l = 0
        freq = defaultdict(int)
        for r in range(len(s)):
            freq[s[r]] = freq.get(s[r], 0) + 1        # admit s[r]
            while (r - l + 1) - max(freq.values()) > k:  # invalid? shrink
                freq[s[l]] -= 1
                l += 1
            longest = max(longest, (r - l + 1))
        return longest
```

Passed 28/28, 30ms (beats 100% — noisy metric; algorithm is optimal regardless).

> **Note:** used `freq.get(s[r], 0) + 1` even though `freq` is a `defaultdict(int)`. With a defaultdict, `freq[s[r]] += 1` alone works (missing keys auto-init to 0). The `.get` form is belt-and-suspenders — correct but redundant once defaultdict is chosen. **Lock one idiom:** pick `defaultdict(int)` + `freq[c] += 1`, OR plain `{}` + `freq[c] = freq.get(c, 0) + 1`. Don't mix.

---

# Why It Works

**Invariant:** at each step `freq` holds the exact character counts of the window `s[l..r]`. The minimum number of replacements to make that window uniform is `window_size - maxfreq`, because the cheapest target is the char already most present. The window is valid (achievable within budget) while `window_size - maxfreq <= k`.

**Optimality of maxfreq:** targeting char `X` costs `size - count(X)`. `size` is fixed per window, so cost is minimized by maximizing `count(X)` → choose the most frequent char. Any other choice is strictly more work.

---

# Traversal Logic

How we move through the data:

- `r` marches left → right, one pass, admitting each char into `freq`.
- `l` only advances when the window becomes invalid, and only forward (never rewinds).

```
"A A B A B B A"   k=1
 l                  window grows while size - maxfreq <= k
       r-> invalid at some point -> shrink l forward
both pointers move RIGHT only
```

---

# Business Logic

The decision at each step:

1. **Admit:** `freq[s[r]] += 1` — bring the new char into the window's counts.
2. **Validate / shrink:** `while (r-l+1) - max(freq.values()) > k:` drop `freq[s[l]]`, advance `l` — until replacements needed fits the budget.
3. **Measure:** `longest = max(longest, r - l + 1)`.

---

# Variable Roles

| Variable | Represents | Stores | How It Changes | Why It Exists |
|----------|------------|--------|----------------|---------------|
| `freq` | char counts in the window | dict (char → int) | `+1` on admit, `-1` on evict | gives maxfreq → replacement cost |
| `l` | left edge of window | int (index) | `+1` only when window invalid | marks valid window start |
| `r` | right edge / current char | int (index) | `+1` each loop (the `for`) | drives the single pass |
| `longest` | best valid window length | int | `max()` each pass | the running answer |

---

# Visual Walkthrough

Input `"AABABBA"`, k=1, expected `4`:

| r | s[r] | freq after admit | size | maxfreq | need=size−maxfreq | >k? | action | l | longest |
|---|------|------------------|------|---------|-------------------|-----|--------|---|---------|
| 0 | A | {A:1} | 1 | 1 | 0 | no | — | 0 | 1 |
| 1 | A | {A:2} | 2 | 2 | 0 | no | — | 0 | 2 |
| 2 | B | {A:2,B:1} | 3 | 2 | 1 | no | — | 0 | 3 |
| 3 | A | {A:3,B:1} | 4 | 3 | 1 | no | — | 0 | **4** |
| 4 | B | {A:3,B:2} | 5 | 3 | 2 | YES | shrink ×2 | 2 | 4 |
| 5 | B | {A:1,B:3} | 4 | 3 | 1 | no | — | 2 | 4 |
| 6 | A | {A:2,B:3} | 5 | 3 | 2 | YES | shrink ×2 | 4 | 4 |

At r=4: shrink fires twice (drop s[0]=A → {A:2,B:2} still invalid → drop s[1]=A → {A:1,B:2} valid). The multi-step shrink is exactly why it's a `while`, not an `if`. Returns **4**. ✓

---

# Optimization Journey

1. **Brute Force:** for every substring, count chars, check `size - maxfreq <= k` → O(n²) windows × O(n) counting.
2. **Bottleneck:** recomputing counts from scratch for every window.
3. **Observation:** as the window slides, counts change by ±1; the validity check is just `size - maxfreq` against `k`.
4. **Optimization:** maintain `freq` incrementally; expand right, shrink left when invalid.
5. **Final Approach:** one pass, `l` advances ≤ n times → O(n) (maxfreq over ≤26 keys is O(1)).

---

# Pattern Recognition

## Signal Phrases

- "longest substring" + "replace up to k characters" + "same letter" → variable window with a *tolerance budget*
- "up to k changes" → track a count, compare `size − dominant` against `k` (not set membership)

## Pattern Used

Sliding Window (variable size) + frequency dict.

## Why This Pattern Fits

- Window represents the current candidate substring; grows and shrinks.
- A *counting* structure (not a set) is needed because the decision depends on the most frequent char's count.
- Both pointers move forward only → linear.

---

# Pattern Comparison

| vs. Problem 3 | Problem 3 (Longest Substring No Repeats) | Problem 424 (this) |
|---------------|------------------------------------------|--------------------|
| Structure | `set` (membership) | `dict` (counts) |
| Validity | no duplicate in window | `size − maxfreq ≤ k` |
| Shrink trigger | `s[r]` already present | replacements needed exceed budget |
| Same skeleton? | yes | yes — `for r: admit; while invalid: shrink; measure` |

---

# Common Mistakes

- Modeling the budget as a decrementing `swapsleft` counter instead of recomputing `size - maxfreq` per window (my Attempt 1).
- Comparing `s[l] == s[r]` (endpoint equality) — wrong question for this problem.
- Using a `set` instead of a count-capable dict.
- `if` instead of `while` on the shrink (the window can need multiple evictions — see r=4).
- Mixing dict idioms (`.get` on a defaultdict is redundant; pick one).

---

# Edge Cases

| Case | Input | k | Expected | How Handled |
|------|-------|---|----------|-------------|
| All same | `"AAAA"` | 0 | 4 | size − maxfreq = 0 always, never shrinks |
| k ≥ length | `"ABCD"` | 4 | 4 | whole string fits budget |
| k = 0 | `"ABAB"` | 0 | 1 | any second distinct char triggers shrink |
| Single char | `"A"` | 1 | 1 | one pass, length 1 |

---

# Debugging Lessons

- **Wrong model > buggy code in severity.** Attempt 1's bugs (infinite loop, window collapse) were downstream of anchoring on endpoint comparison instead of window frequencies. Fixing the model fixed everything at once.
- **A 0 left in the count dict is harmless under `max()`** — reasoned this out rather than defensively deleting keys.
- **Syntax vocabulary is the current bottleneck, not algorithms.** Multiple failed submits + googling `.get` were all mechanical. Mitigation: memorize one dict-counting idiom cold; do a "does each line actually run" pass before submitting, separate from the logic trace.

---

# NeetCode / Official Solution

(Included because I pasted it in chat.)

## Code

```python
class Solution:
    def characterReplacement(self, s: str, k: int) -> int:
        count = {}
        res = 0
        l = 0
        for r in range(len(s)):
            count[s[r]] = 1 + count.get(s[r], 0)
            while (r - l + 1) - max(count.values()) > k:
                count[s[l]] -= 1
                l += 1
            res = max(res, r - l + 1)
        return res
```

## Comparison to My Solution

Functionally identical. The only difference is the counting structure: NeetCode uses a plain `count = {}` with `count.get(s[r], 0)`, while I used `defaultdict(int)`. Both O(n)/O(1)-space (≤26 keys), same shrink condition, same skeleton. NeetCode's plain-dict version is the reason the `.get` idiom appears — with my defaultdict the `.get` was unnecessary. Nothing algorithmic to learn from the diff; it's an idiom choice.

> NeetCode also teaches an optimization where you track `maxfreq` as a running variable and never recompute or decrease it (the window only grows). It's correct for this problem but relies on a subtle argument — deliberately skipped in favor of the clear honest-shrink version.

---

# Deep Understanding

The non-obvious part is that `size - maxfreq` is not just *a* way to measure replacements — it's the *minimum*, because keeping the most frequent char is provably optimal (maximizing what you keep minimizes what you replace). That's what lets a single scalar comparison (`size - maxfreq <= k`) decide window validity. The model breakthrough was switching the question from "are these two chars equal" to "how cheaply can this whole window become uniform."

---

# Complexity Analysis

| Metric | Complexity | Reason |
|--------|------------|--------|
| Time | O(n) | one pass; `max(freq.values())` is over ≤26 keys = O(1); `l` advances ≤ n total |
| Space | O(1) | dict holds at most 26 uppercase letters |

---

# Related Problems

| # | Problem | Difficulty | Same Pattern? | Notes |
|---|---------|------------|---------------|-------|
| 3 | Longest Substring Without Repeating Chars | Medium | yes (variable window) | set vs dict — membership not counts |
| 567 | Permutation in String | Medium | yes (fixed window) | reuses freq counting; checks two maps match |
| 76 | Minimum Window Substring | Hard | yes (variable window) | counts again, but shrink to minimize |
| 1004 | Max Consecutive Ones III | Medium | yes (variable window) | same `size − (kept) ≤ k` shape, binary |

---

# Interview Explanation (30 Seconds)

Variable sliding window with a frequency dict. For each window, the minimum replacements to make it uniform is `window_size - count_of_most_frequent_char`. Expand right; when that value exceeds `k`, shrink left. Track the longest valid window. One pass, O(n).

---

# Interview Explanation (2 Minutes)

Recognition: "longest substring after up to k replacements to make all chars the same" → variable window with a tolerance budget. I keep a dict of char counts in the window `[l, r]`. The invariant is that the cheapest way to make any window uniform is to keep its most frequent char and replace everything else, so the replacement cost is `(r - l + 1) - max(freq.values())`. For each `r` I admit `s[r]`, then while that cost exceeds `k` I shrink from the left (decrement the leaving char, advance `l`), and record the window length against the max. Why max frequency is optimal: cost is `size - count(target)`, size is fixed, so minimizing cost means maximizing the kept char. Complexity is O(n) — `l` only moves forward at most n times total, and the max over the dict is O(26) = O(1). The one design choice that makes it correct is the `while` on the shrink, since exceeding budget can require evicting several chars.

---

# Common Follow-Up Questions

- **Why is `size - maxfreq` the *minimum* replacements?** → keeping the most frequent char leaves the fewest to replace; any other target costs more.
- **Can you avoid recomputing `max` every step?** → track `maxfreq` as a running variable that only increases; the window never needs to shrink below the best seen. (Subtle but valid.)
- **What if the alphabet were huge (Unicode)?** → `max(freq.values())` becomes O(alphabet); the running-maxfreq trick or a count-of-counts structure helps.
- **Lowercase + uppercase, or arbitrary chars?** → same code, dict just holds more keys; still bounded by distinct chars in the window.

---

# Key Takeaways

- When a window problem allows "up to k violations," track a *count* and compare `size − dominant` against `k` — not set membership.
- `size − maxfreq` is the minimum replacement cost because keeping the most frequent char is optimal.
- Wrong model produces cascading bugs; fixing the model (endpoint-compare → frequency-count) fixed all of them.
- Current growth edge is Python syntax fluency, not algorithms — memorize the counting idiom, dry-run for syntax before submitting.

---

# Revision Notes

**1 week:** re-derive *why* it's `maxfreq` (minimize leftover) rather than recalling the formula. Re-trace `"AABABBA"` k=1 → 4, watching the double-shrink at r=4.

**1 month:** recall the one-liner — replacements needed = `size − maxfreq`, valid while ≤ k. And the skeleton: admit right, shrink left while invalid, measure.

---

# Final Mental Model

You're sliding a stretchy window over the string, and inside it you keep a tally of how many of each letter you have. At any moment the "cost" to make the window all-one-letter is: total letters minus your biggest pile (you keep the biggest pile, repaint the rest). As long as that repaint cost stays within your budget `k`, you let the window grow. The instant it'd cost more than `k`, you slide the left edge forward, dropping letters off the back, until you're back within budget. The widest the window ever got is your answer. The one trick to remember: always keep the biggest pile — that's what makes the repaint cheapest.
