# 567. Permutation in String

## Problem

Given two strings `s1` and `s2` (lowercase only), return `True` if `s2` contains a permutation of `s1` as a substring, else `False`. A permutation of `s1` is any rearrangement of its characters.

**Example:** `s1 = "abc"`, `s2 = "lecabee"` → `True` (the substring `"cab"` is a permutation of `"abc"`).

---

# My Learning Journey

Reached for the *optimal* approach unprompted — two fixed-size 26-int frequency arrays rather than dicts — which is the better version of this problem, not the beginner one. The core idea (slide a window of `len(s1)` over `s2`, compare frequency arrays) was right from the start. The bugs were all execution polish, not concept.

First attempt had three issues: `ord(a)` instead of `ord('a')` (a `NameError` — `a` isn't a defined variable), the match check sitting *outside* the loop so it only ran once at the very end, and the shrink gated behind the match condition (`while countS2 != countS1`) instead of being a pure size clamp. The fix was structural: separate the two concerns — *maintaining* a fixed-size window vs *checking* if it matches.

Second attempt fixed all three but left a fragile `and l < r` on the clamp condition. Dropped it — for `len(s1) >= 1` the size check is self-limiting and `l < r` is redundant-to-harmful. Kept the shrink as a `while` though it can only ever fire once (window overflows by at most 1 per step, since one char enters per iteration); an `if` expresses the invariant more clearly but `while` is equally correct here.

The concept check sealed it. Two ideas I had to state in my own words:

- **Why fixed-size, not variable:** in problems 3 and 424 the window grew/shrank on a *content* rule (no duplicate / replacement budget). Here the shrink is pure *size* — `s1` has a fixed length, so the answer must be exactly that wide; the instant the window exceeds `len(s1)`, clamp it. Constant-width slide, not content-driven resize.
- **Why matching count arrays detects a permutation:** a permutation is a rearrangement — same characters, same quantities, different order. Two strings are permutations iff they have the identical multiset of characters. Counting *erases order*, and order is the only thing a permutation is allowed to change — so the count array is the permutation-invariant fingerprint. `"abc"`, `"cab"`, `"bca"` all collapse to the same array.

---

# Attempt 1 (three bugs)

```python
class Solution:
    def checkInclusion(self, s1: str, s2: str) -> bool:
        countS1 = [0]*26
        countS2 = [0]*26
        for c in s1:
            countS1[ord(c) - ord(a)] += 1          # bug: ord(a) -> NameError
        l = 0
        for r in range(len(s2)):
            countS2[ord(s2[r]) - ord(a)] += 1
            while countS2 != countS1 and l < r:    # bug: shrink gated on match
                if (r - l + 1) > len(s1):
                    countS2[ord(s2[l]) - ord(a)] -= 1
                    l += 1
        return countS2 == countS1                  # bug: check outside loop, runs once
```

| Bug | Type | Fix |
|-----|------|-----|
| `ord(a)` | NameError | `ord('a')` — it's the character, needs quotes |
| `while countS2 != countS1` drives the shrink | Logic | fixed window shrinks on *size*, unconditionally — not gated on match |
| `return countS2 == countS1` outside the loop | Logic | a permutation can appear anywhere in `s2` and slide away; must check inside the loop every step and return True on hit |

---

# Accepted Solution

```python
class Solution:
    def checkInclusion(self, s1: str, s2: str) -> bool:
        countS1 = [0]*26
        countS2 = [0]*26
        for c in s1:
            countS1[ord(c) - ord('a')] += 1
        l = 0
        for r in range(len(s2)):
            countS2[ord(s2[r]) - ord('a')] += 1       # admit right
            while (r - l + 1) > len(s1):              # size clamp (fires ≤ once)
                countS2[ord(s2[l]) - ord('a')] -= 1   # evict left
                l += 1
            if countS2 == countS1:                    # window is now len(s1)-wide
                return True
        return False
```

Passed 31/31, 32ms.

> **Note:** the `while` clamp can only ever execute once per iteration (one char enters → at most one leaves). An `if` would express that invariant more clearly; `while` is equally correct. Also: the earlier `and l < r` guard was dropped — for `len(s1) >= 1` the size comparison is self-limiting.

---

# Why It Works

**Invariant:** `countS2` always holds the character counts of the current window in `s2`, and after the clamp the window is exactly `len(s1)` characters wide. `countS1` holds the (fixed) counts of `s1`. The window is a permutation of `s1` iff `countS2 == countS1` — identical multisets.

**Key operation:** counting erases order. Since a permutation only rearranges (never changes which/how many characters), equal count arrays is a necessary and sufficient test for "this window is a permutation of `s1`."

---

# Traversal Logic

How we move through the data:

- `r` marches left → right over `s2`, admitting each char into `countS2`.
- After each admit, clamp: if the window exceeds `len(s1)`, evict the leftmost char and advance `l`. The window slides at constant width.

```
s1 = "abc"  (width 3)
s2 = l e c a b e e
     [   ]              window held at width 3, slides right one step at a time
       [   ]
         [   ] <- "cab" here: counts match -> return True
```

---

# Business Logic

The decision at each step:

1. **Admit:** `countS2[idx(s2[r])] += 1`.
2. **Clamp:** while window width `> len(s1)`, `countS2[idx(s2[l])] -= 1`; `l += 1`.
3. **Check:** if `countS2 == countS1`, return `True`.
4. After the loop, return `False`.

---

# Variable Roles

| Variable | Represents | Stores | How It Changes | Why It Exists |
|----------|------------|--------|----------------|---------------|
| `countS1` | char counts of `s1` (target) | list[26] | built once, never changes | the fingerprint to match against |
| `countS2` | char counts of current window | list[26] | `+1` admit, `-1` evict | the window's fingerprint |
| `l` | left edge of window | int (index) | `+1` on clamp | keeps window at width `len(s1)` |
| `r` | right edge / current char | int (index) | `+1` each loop | drives the slide |

`idx(ch) = ord(ch) - ord('a')` maps `'a'..'z'` → `0..25`.

---

# Visual Walkthrough

`s1 = "ab"` (counts: a:1, b:1), `s2 = "eidbaooo"`, expected `True`:

| r | s2[r] | countS2 (a,b,...) after admit | width | clamp? | window | match? |
|---|-------|-------------------------------|-------|--------|--------|--------|
| 0 | e | e:1 | 1 | no | "e" | no |
| 1 | i | e:1,i:1 | 2 | no | "ei" | no |
| 2 | d | e:1,i:1,d:1 | 3 | yes → drop e | "id" | no |
| 3 | b | i:1,d:1,b:1 | 3 | yes → drop i | "db" | no |
| 4 | a | d:1,b:1,a:1 | 3 | yes → drop d | "ba" → a:1,b:1 | **YES → True** |

Returns `True` at r=4. ✓

---

# Optimization Journey

1. **Brute Force:** for every length-`len(s1)` substring of `s2`, sort it (or count it) and compare to `s1` → O(n · m log m) or O(n · m).
2. **Bottleneck:** recounting each substring from scratch.
3. **Observation:** consecutive windows differ by one char in, one char out — update counts incrementally.
4. **Optimization:** slide a fixed-width window, maintaining `countS2` in O(1) per step; compare arrays.
5. **Final Approach:** O(n) passes, O(26) array compare per step → O(26n) = O(n).

---

# Pattern Recognition

## Signal Phrases

- "permutation of s1 as a substring" → fixed-size window of width `len(s1)`
- "permutation" / "anagram" → frequency-count comparison (order doesn't matter)
- fixed target length → the window size is *given*, so it's a fixed (not variable) window

## Pattern Used

Sliding Window (fixed size) + frequency arrays.

## Why This Pattern Fits

- The answer's width is exactly `len(s1)` — fixed, known up front.
- "Permutation" reduces to "same character multiset" → count arrays.
- Consecutive windows share all but two characters → incremental O(1) updates.

---

# Pattern Comparison

| | 3 (No Repeats) | 424 (Char Replacement) | 567 (this) |
|---|----------------|------------------------|-----------|
| Window type | variable | variable | **fixed** |
| Shrink trigger | duplicate in window | `size − maxfreq > k` | window width `> len(s1)` |
| Structure | set | dict (counts) | two fixed [26] arrays |
| Check | implicit (length) | implicit (length) | explicit array equality |
| Shrink fires | 0..many per step | 0..many per step | 0..1 per step |

---

# Common Mistakes

- `ord('a')` written as `ord(a)` — the latter is an undefined name.
- Gating the shrink on the *match* condition instead of *size* — it's a fixed window, the clamp is purely about width.
- Checking equality only once after the loop instead of every iteration (a permutation can slide in and out).
- Adding an `l < r` guard to the clamp — redundant for `len(s1) >= 1`.

---

# Edge Cases

| Case | s1 | s2 | Expected | How Handled |
|------|----|----|----------|-------------|
| s1 longer than s2 | "abc" | "ab" | False | window never reaches width 3, no match |
| Single char | "a" | "ab" | True | width-1 window, immediate match at r=0 |
| Permutation at end | "ab" | "eidboaba"... | True | check runs every step, catches late matches |
| No match | "ab" | "eidboaoo" | False | array equality never holds, return False |
| Exact equal | "abc" | "abc" | True | window = whole string, counts match |

---

# Debugging Lessons

- **Separate "maintain the window" from "check the window."** Attempt 1 tangled the shrink into the match condition; untangling them fixed the structure.
- **A fixed-size window's clamp is unconditional on size, not gated on content.** This is the defining difference from variable windows.
- **Check the matching condition inside the loop, every step** — a valid window can appear and disappear as you slide.
- **`ord('a')` needs quotes.** Recurring syntax-vocab tax; same class as the `.get` issue on 424.

---

# NeetCode / Official Solution (Optimized — O(1) per-step compare)

This is **not** the version I submitted. My accepted solution (above) compares the full 26-element arrays each step — O(26) per step, correct and clean. NeetCode's version optimizes that comparison down to O(1) using a running `matches` counter. Documented here as the advanced alternative; the version above remains my actual solve.

## Code

```python
class Solution:
    def checkInclusion(self, s1: str, s2: str) -> bool:
        if len(s1) > len(s2):
            return False

        s1Count, s2Count = [0] * 26, [0] * 26
        for i in range(len(s1)):
            s1Count[ord(s1[i]) - ord('a')] += 1
            s2Count[ord(s2[i]) - ord('a')] += 1

        matches = 0
        for i in range(26):
            matches += (1 if s1Count[i] == s2Count[i] else 0)

        l = 0
        for r in range(len(s1), len(s2)):
            if matches == 26:
                return True

            index = ord(s2[r]) - ord('a')          # char ENTERING on the right
            s2Count[index] += 1
            if s1Count[index] == s2Count[index]:
                matches += 1
            elif s1Count[index] + 1 == s2Count[index]:
                matches -= 1

            index = ord(s2[l]) - ord('a')          # char LEAVING on the left
            s2Count[index] -= 1
            if s1Count[index] == s2Count[index]:
                matches += 1
            elif s1Count[index] - 1 == s2Count[index]:
                matches -= 1
            l += 1

        return matches == 26
```

## The Core Idea

Comparing two 26-element arrays every step is O(26). But between consecutive windows, **only two slots change** — the entering char's slot and the leaving char's slot. Every other slot is untouched, so its agreement state can't have changed. So instead of re-checking all 26 each step, track a single number: `matches` = how many of the 26 slots currently agree between `s1Count` and `s2Count`. When a slot changes, only *that slot* can flip its agreement, so you adjust `matches` by at most ±1 in O(1). When `matches == 26`, all slots agree → permutation found.

## The Four if/elif Blocks (the subtle part)

Each character move touches one slot and can do one of three things to that slot's agreement: make it newly equal (+1), break a previous equality (−1), or leave it unequal-to-unequal (no change). The trick is detecting which, using the state *after* the count update plus knowledge of how it changed.

**Char entering (`s2Count[index] += 1`):**

| Condition | Meaning | Action |
|-----------|---------|--------|
| `s1Count[index] == s2Count[index]` | the increment just made this slot equal | `matches += 1` |
| `s1Count[index] + 1 == s2Count[index]` | s2 was equal a step ago, now overshot s1 by 1 → just broke a match | `matches -= 1` |
| neither | still unequal both before and after | no change |

The `+ 1` in the elif reconstructs the *previous* value: `s2Count[index]` is the post-increment value, so `s2Count[index] - 1` was the old value; the slot was equal before iff `s1Count[index] == s2Count[index] - 1`, i.e. `s1Count[index] + 1 == s2Count[index]`.

**Char leaving (`s2Count[index] -= 1`):** mirror logic.

| Condition | Meaning | Action |
|-----------|---------|--------|
| `s1Count[index] == s2Count[index]` | the decrement just made this slot equal | `matches += 1` |
| `s1Count[index] - 1 == s2Count[index]` | s2 was equal a step ago, now undershot s1 by 1 → just broke a match | `matches -= 1` |
| neither | still unequal | no change |

## Structural Differences from My Version

| Aspect | My accepted version | NeetCode optimized |
|--------|---------------------|--------------------|
| Per-step compare | `s2Count == s1Count` → O(26) | `matches == 26` → O(1) |
| Window mechanics | admit right, clamp by size | explicit enter-right + leave-left every step (window pre-sized) |
| Initial fill | counts built as window grows | both counts pre-filled for first `len(s1)` chars, then `matches` initialized once |
| Loop range | `range(len(s2))` with a size clamp | `range(len(s1), len(s2))` — window starts already full |
| Total time | O(26n) | O(n) |

## Bug Traps in the Optimized Version

- The `elif` conditions check the *pre-change* equality by reconstructing the old value (`+1` for an increment, `-1` for a decrement). Get the sign wrong and `matches` drifts silently → wrong answer with no crash.
- The initial `matches` loop must run *after* both counts are pre-filled, or you start with a wrong baseline.
- The `if matches == 26: return True` sits at the *top* of the loop so the pre-filled first window (before any slide) gets checked. Easy to misplace.

**Verdict:** my array-compare version is easier to write correctly and is what I'd reach for first; this `matches` version is the answer to "make the comparison O(1)" and is worth being able to derive, but its four-branch bookkeeping is exactly the kind of thing that's easy to get subtly wrong under interview pressure.

---

# Deep Understanding

The crux is that "is a permutation of" is equivalent to "has the same character multiset," because a permutation changes *only* order. Counting throws away order and preserves exactly the multiset, so equal count arrays is a sound and complete permutation test. Everything else — the fixed window, the incremental updates — is just an efficient way to ask that question at every position in `s2`.

---

# Complexity Analysis

| Metric | Complexity | Reason |
|--------|------------|--------|
| Time | O(n) | one pass over s2; each array compare is O(26) = O(1) → O(26n) |
| Space | O(1) | two fixed 26-element arrays regardless of input size |

(`n = len(s2)`. The O(26) per-step compare is the only non-trivial constant; an O(1)-compare variant exists — see below.)

---

# Related Problems

| # | Problem | Difficulty | Same Pattern? | Notes |
|---|---------|------------|---------------|-------|
| 242 | Valid Anagram | Easy | counting (no window) | same "equal counts = permutation" idea, whole strings |
| 438 | Find All Anagrams in a String | Medium | yes (fixed window) | identical to 567 but collect all start indices |
| 76 | Minimum Window Substring | Hard | variable window + counts | counts again, but variable + minimize |
| 424 | Longest Repeating Char Replacement | Medium | variable window + counts | previous problem; size − maxfreq |

---

# Interview Explanation (30 Seconds)

Fixed-size sliding window. I build a frequency array for `s1`, then slide a window of width `len(s1)` across `s2`, maintaining its frequency array incrementally — add the entering char, drop the leaving char. At each position, if the two arrays match, the window is a permutation of `s1`, return True. O(n) time, O(1) space.

---

# Interview Explanation (2 Minutes)

Recognition: "contains a permutation of s1 as a substring." A permutation is just a reordering, so two strings are permutations iff they share the same character multiset — which I capture with a 26-element count array, since counting discards order and keeps only quantities. The answer's width is fixed at `len(s1)`, so this is a fixed-size window, not a variable one: I slide a window of exactly that width over `s2`, and the only window maintenance is clamping the size — add the entering char on the right, and if the window grew past `len(s1)`, drop the leaving char on the left. After each step the window is exactly `len(s1)` wide, so I compare its count array to `s1`'s; if equal, return True. Complexity is O(n) — one pass, and the array compare is O(26) = O(1). If pushed to make the compare itself O(1), I'd track a single `matches` counter for how many of the 26 positions currently agree, updating it by ±1 as characters enter and leave, and return True when `matches == 26`.

---

# Common Follow-Up Questions

- **Make the per-step comparison O(1)?** → track a `matches` count (positions where both arrays agree), update ±1 on each enter/leave, success when `matches == 26`.
- **Return all start indices, not just a boolean?** → that's problem 438; same window, append `l` whenever it matches instead of returning.
- **Larger / Unicode alphabet?** → swap the 26-array for a dict; the compare becomes O(distinct chars).
- **What if s1 longer than s2?** → window never reaches the required width; returns False naturally.

---

# Key Takeaways

- A *fixed*-size window clamps on **size**, unconditionally — distinct from variable windows that resize on content.
- "Permutation / anagram" → compare character counts; counting erases the order that a permutation is allowed to change.
- Check the match condition *inside* the loop every step — valid windows slide in and out.
- The two-array compare is O(26)=O(1); a `matches`-counter variant makes it truly O(1) if needed.

---

# Revision Notes

**1 week:** state the two definitions — fixed window clamps on size; permutation = equal multiset (counting erases order). Re-trace `s1="ab" s2="eidbaooo"` → True.

**1 month:** recall the skeleton — build s1 counts, slide width-`len(s1)` window over s2 maintaining counts incrementally, compare each step.

---

# Final Mental Model

Think of `s1`'s character counts as a lock with 26 tumblers set to specific heights. You drag a fixed-width frame across `s2`, and inside the frame you keep a live count of its 26 tumblers — every time the frame moves one step, one character enters and one leaves, so you bump two tumblers. At every position you ask: do my 26 tumblers exactly match the lock? If yes, the frame holds a permutation of `s1` and the lock opens. The frame never changes width — it just slides, because the thing you're hunting for is always exactly `len(s1)` characters long.
