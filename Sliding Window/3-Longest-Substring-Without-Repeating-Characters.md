# 3. Longest Substring Without Repeating Characters

## Problem

Given a string `s`, return the length of the longest substring that contains no duplicate characters. A substring is a contiguous run of characters.

**Example:** `s = "zxyzxyz"` → `3` (the substring `"xyz"` is the longest with all-unique characters).

---

# My Learning Journey

Reached for the right shape on my own this time — set for O(1) duplicate checks, two pointers `l`/`r` as the window edges, `for r` as the outer march. The pattern recognition was immediate, which is a step up from 121 where I started in the wrong shape entirely.

The bug was in the *shrink*. My first version used an `if s[r] in windowSet` instead of a `while`, removed `s[l]` exactly once, and hid the `add(s[r])` inside an `else` so the new char never got added on a duplicate step. That desynced the set from the actual window. Tracing `"abba"` exposed it — it returned 3 (claiming `"bba"` was valid) because at the first `b` collision I evicted `a` instead of shrinking until the duplicate `b` was gone, and the set ended up lying about the window contents.

The fix was a single structural swap: `if` → `while` for the shrink (because the duplicate might require evicting several left chars, not one), and pull `add(s[r])` out of the `else` so it runs unconditionally after the window is made valid. After that it passed first try — no mechanical bug, no resubmit loop. First problem this session where the hint→fix→accept was clean.

The concept check at the end is where it actually locked in. I had to explain the invariant in my own words:

- **Why `l` never rewinds:** the window `[l..r]` is always the longest valid (dupe-free) substring *ending at r*. Anything starting left of `l` would have to re-include the duplicate we just evicted, so it's automatically invalid — there's no longer valid substring hiding back there to miss.
- **Why it's O(n) not O(n²):** `l += 1` can run at most `n` times across the *entire* run, because `l` starts at 0 and only ever increases, so it can reach `n` at most. The `for` is `n` steps and the `while` does at most `n` total removals summed over all iterations — not `n` per iteration. `n + n = O(n)`. (This is amortized analysis — charging the inner loop against a one-directional bounded counter.)

---

# Attempt 1

```python
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        l = 0
        longest = 0
        windowSet = set()
        for r in range(len(s)):
            if s[r] in windowSet:
                windowSet.remove(s[l])
                l += 1
            else:
                windowSet.add(s[r])
                r + 1                      # dead line, does nothing
            window = (r - l) + 1
            longest = max(window, longest)
        return longest
```

**What went wrong:**

| Bug | Type | Explanation |
|-----|------|-------------|
| `if` instead of `while` | Logic | a duplicate may require evicting several left chars; one `remove` doesn't guarantee the dupe is gone |
| removes `s[l]` blindly | Logic | evicts the left edge, not necessarily the char causing the collision → set desyncs from window |
| `add` hidden in `else` | Logic | on a duplicate step, `s[r]` is never added after making room — set and window disagree |
| `r + 1` | Recurring #8 | mutating/incrementing the loop var inside `for r in range()` is pointless (and here it's not even an assignment) — the range overwrites `r` next pass anyway |

**Trace that exposed it — `"abba"`, expected `2`:**

| r | s[r] | in set? | action | windowSet | l | len |
|---|------|---------|--------|-----------|---|-----|
| 0 | a | no | add a | {a} | 0 | 1 |
| 1 | b | no | add b | {a,b} | 0 | 2 |
| 2 | b | yes | remove s[0]=a, l=1 | {b} | 1 | 2 |
| 3 | a | no | add a | {b,a} | 1 | **3** ❌ |

Returned `3`, but `s[1:4] = "bba"` has two b's. The set `{b,a}` was a lie — at r=2 the wrong char was evicted.

---

# Accepted Solution(s)

```python
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        l = 0
        longest = 0
        windowSet = set()
        for r in range(len(s)):
            while s[r] in windowSet:        # shrink until s[r] has room
                windowSet.remove(s[l])
                l += 1
            windowSet.add(s[r])             # always, after shrinking
            window = (r - l) + 1
            longest = max(window, longest)
        return longest
```

Passed 32/32, 27ms (beats 100% — noisy metric, but the algorithm is optimal anyway).

---

# Why It Works

**Invariant:** at every step, `windowSet` holds exactly the characters in `s[l..r]`, and that window is the **longest duplicate-free substring ending at index r**.

**Key operation:** before adding `s[r]`, shrink from the left until `s[r]` is no longer in the window. This guarantees the new char can be added without creating a duplicate, and that the window remains the maximal valid one ending at `r`.

The global answer is the max of these per-position window lengths — because the optimal substring ends at *some* index, and at that index the algorithm held the maximal valid window.

---

# Traversal Logic

How we move through the data:

- `r` marches left → right, one pass, every character entered once.
- `l` only crawls forward, and only when a duplicate forces it. It never resets or rewinds.

```
s = "a b c a b c b b"
     l                       window grows: a, ab, abc
       r-> hits 'a' dup at index 3
     -> shrink l past first 'a', window = "bca"
l and r both only ever move RIGHT
```

---

# Business Logic

The decision at each step:

1. **Shrink:** `while s[r] in windowSet: remove s[l]; l += 1` — evict from the left until the incoming char has room.
2. **Admit:** `windowSet.add(s[r])` — always, after shrinking.
3. **Measure:** `longest = max(longest, r - l + 1)` — record this window's length.

---

# Variable Roles

| Variable | Represents | Stores | How It Changes | Why It Exists |
|----------|------------|--------|----------------|---------------|
| `windowSet` | chars currently in the window | set | `.add` on entry, `.remove` on shrink | O(1) duplicate check |
| `l` | left edge of window | int (index) | `+1` only when a dupe is hit | marks where the valid window starts |
| `r` | right edge / current char | int (index) | `+1` each loop (the `for`) | drives the single pass |
| `window` | current window length | int | recomputed each pass | scratch feeding the max |
| `longest` | best length seen | int | `max()` each pass | the running answer |

---

# Visual Walkthrough

Input `"abcabcbb"`, expected `3`:

| r | s[r] | while shrinks? | windowSet after | l | r-l+1 | longest |
|---|------|----------------|-----------------|---|-------|---------|
| 0 | a | no | {a} | 0 | 1 | 1 |
| 1 | b | no | {a,b} | 0 | 2 | 2 |
| 2 | c | no | {a,b,c} | 0 | 3 | **3** |
| 3 | a | yes: rm a, l=1 | {b,c,a} | 1 | 3 | 3 |
| 4 | b | yes: rm b, l=2 | {c,a,b} | 2 | 3 | 3 |
| 5 | c | yes: rm c, l=3 | {a,b,c} | 3 | 3 | 3 |
| 6 | b | yes: rm a,b → l=5 | {c,b} | 5 | 2 | 3 |
| 7 | b | yes: rm c,b → l=7 | {b} | 7 | 1 | 3 |

Returns **3**. ✓ Note r=6 and r=7 show the `while` evicting *multiple* chars in one step — exactly the case the `if`-version couldn't handle.

---

# Optimization Journey

1. **Brute Force:** check every substring for uniqueness → O(n²) substrings × O(n) check = O(n³), or O(n²) with a per-start set.
2. **Bottleneck:** re-scanning overlapping substrings from scratch.
3. **Observation:** if `s[l..r]` is dupe-free, extending to `r+1` only fails if `s[r+1]` is already inside — and the fix is to shrink `l` forward, never restart.
4. **Optimization:** maintain a single window with a set; expand right, shrink left on collision.
5. **Final Approach:** one pass, `l` advances ≤ n times total → O(n).

---

# Pattern Recognition

## Signal Phrases

- "longest substring" + "without duplicate characters" → variable-size window
- "contiguous" → window, not subsequence
- constraint that can be *violated and repaired* (a dupe) → expand-right / shrink-left

## Pattern Used

Sliding Window (variable size) + Hash Set.

## Why This Pattern Fits

- The window represents the current valid substring; it grows and shrinks.
- A set gives O(1) "is this char in the window" checks.
- Both pointers move forward only → linear.

---

# Common Mistakes

- Using `if` instead of `while` for the shrink (only removes one char; fails when the dupe is deep in the window).
- Removing `s[l]` without checking whether the actual duplicate is gone yet — handled correctly *because* the `while` re-tests `s[r] in windowSet` each pass.
- Adding `s[r]` conditionally (in an `else`) instead of unconditionally after the shrink.
- Incrementing the `for` loop variable by hand (recurring bug #8).

---

# Edge Cases

| Case | Input | Expected | How Handled |
|------|-------|----------|-------------|
| All same | `"bbbb"` | 1 | each new b shrinks to single-char window |
| All unique | `"abcdef"` | 6 | while never fires, window grows to full length |
| Empty | `""` | 0 | loop never runs, returns 0 |
| Single char | `"a"` | 1 | one pass, length 1 |
| Dupe far apart | `"abba"` | 2 | while evicts multiple chars at the second collision |

---

# Debugging Lessons

- **`if` vs `while` on a shrink is a logic bug, not a style choice** — if the violation can require more than one repair step, it must be a `while`.
- **Keep the auxiliary structure (set) and the window (l..r) in lockstep.** The Attempt-1 bug was the set claiming membership that the window didn't reflect. Unconditional `add` after the shrink keeps them synced.
- **Trace a string with a far-apart duplicate (`"abba"`)** — it's the input that separates a correct `while`-shrink from a broken single-`remove`.

---

# NeetCode / Official Solution

(Included because I pasted it in chat.)

## Code

```python
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        charSet = set()
        l = 0
        res = 0
        for r in range(len(s)):
            while s[r] in charSet:
                charSet.remove(s[l])
                l += 1
            charSet.add(s[r])
            res = max(res, r - l + 1)
        return res
```

## Comparison to My Solution

Functionally identical to my accepted version. Only differences are cosmetic: `charSet`/`res` naming vs my `windowSet`/`longest`, and `res = max(res, r - l + 1)` written inline vs my two-line `window = (r-l)+1; longest = max(...)`. Same algorithm, same O(n)/O(n) complexity, same structure. Nothing to learn from the diff — I'd already converged on the canonical form.

---

# Deep Understanding

The non-obvious part is that the `while` inside the `for` is *not* quadratic. `l` is a one-way ratchet — it can advance a total of `n` times across the whole run, period. So the inner loop's total work over the entire algorithm is bounded by `n`, independent of how it's distributed across iterations. This amortized argument is the reusable insight for nearly every two-pointer / sliding-window problem.

---

# Complexity Analysis

| Metric | Complexity | Reason |
|--------|------------|--------|
| Time | O(n) | `for` does n steps; `while` does ≤ n total removals (amortized) |
| Space | O(min(n, charset)) | the set holds at most one of each distinct character |

---

# Related Problems

| # | Problem | Difficulty | Same Pattern? | Notes |
|---|---------|------------|---------------|-------|
| 424 | Longest Repeating Character Replacement | Medium | yes (variable window) | shrink condition uses window size − max freq, not a set |
| 567 | Permutation in String | Medium | yes (fixed window) | fixed-size window + freq match |
| 76 | Minimum Window Substring | Hard | yes (variable window) | shrink to *minimize* instead of maximize |
| 121 | Best Time to Buy/Sell Stock | Easy | window (implicit edge) | previous problem; left edge as a value not an index |

---

# Interview Explanation (30 Seconds)

Variable sliding window with a set. Expand the right edge one char at a time; whenever the new char is already in the window, shrink from the left until it isn't, then add it. Track the max window length. One pass, O(n) time.

---

# Interview Explanation (2 Minutes)

Recognition: "longest contiguous substring under a violatable constraint" → variable window. I keep a set of the chars currently in the window `[l, r]`. The invariant is that this window is always the longest dupe-free substring ending at `r`. For each `r`, if `s[r]` is already in the set I shrink from the left — removing `s[l]` and advancing `l` — until `s[r]` has room, then add it and record `r - l + 1` against the running max. The subtle point is complexity: the nested `while` looks O(n²) but `l` only ever moves forward and at most `n` times total, so the whole thing is amortized O(n). Space is O(min(n, alphabet)) for the set. The one design decision that makes it correct is the `while` (not `if`) on the shrink — a single removal isn't enough when the duplicate sits deep in the window.

---

# Common Follow-Up Questions

- **Return the substring itself, not just length?** → track the `(l, r)` pair when the max updates, slice at the end.
- **What about a fixed alphabet — can you drop the set?** → use a fixed-size array/`dict` of last-seen indices and jump `l` directly past the previous occurrence (avoids the inner `while`).
- **Optimize the shrink to O(1) jumps?** → store last-seen index of each char in a dict; on collision set `l = max(l, last[s[r]] + 1)`. Still O(n), fewer operations.
- **Unicode / huge alphabet?** → set/dict scales with distinct chars seen, so space is bounded by the window, not a fixed 26/128.

---

# Key Takeaways

- The canonical variable-window skeleton: `for r: while invalid: shrink l; admit r; measure`.
- A shrink that can need multiple steps is a `while`, never an `if`.
- Nested `while`-in-`for` is O(n) when the inner pointer is a one-way ratchet — amortized analysis.

---

# Revision Notes

**1 week:** re-state the invariant out loud — "window is the longest dupe-free substring *ending at r*." Re-trace `"abba"` → 2 and confirm the `while` evicts multiple chars.

**1 month:** just recall the skeleton: expand right, shrink left while invalid, measure. And the O(n) reason: `l` advances ≤ n times total.

---

# Final Mental Model

Picture reading a string left to right with a stretchy window. You keep pulling the right edge forward, adding each new letter to a bag of "what's currently in my window." The moment you'd add a letter that's already in the bag, you slide the left edge forward — dropping letters out of the bag one at a time — until the offending letter is gone, then drop the new one in. You never slide the left edge backward, because anything behind it would just re-introduce a letter you already kicked out. The longest the window ever got is your answer.
