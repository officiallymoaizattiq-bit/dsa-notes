# 125. Valid Palindrome

## Problem

Given a string `s`, return `True` if it's a palindrome after lowercasing it and removing all non-alphanumeric characters. A palindrome reads the same forward and backward.

Concrete example:

```
Input:  "A man, a plan, a canal: Panama"
Cleaned: "amanaplanacanalpanama"
Output: True
```

```
Input:  "race a car"
Cleaned: "raceacar"
Output: False
```

---

# My Learning Journey

i came in with the core two-pointer skeleton already correct — converging `start`/`end`, compare, step inward. the algorithm shape was never the problem. what i actually learned on this one was a chain of python-specific gotchas, and i hit them one at a time in sequence.

the first miss was conceptual: my opening attempt compared raw characters with no normalization. it died instantly on `"A man, a plan, a canal: Panama"` because `'A' != 'a'` and because spaces/commas were still in the comparison. so the real lesson here wasn't "two pointers" — it was "clean the data first, or skip the junk as you go."

i chose the skip-in-loop route (the better one, O(1) space), and that's where the python bugs stacked up. i learned four distinct things the hard way:

1. **strings are immutable** — i wrote `s.lower()` on its own line and assumed it mutated `s`. it doesn't. it returns a new string i was throwing away.
2. **`isalnum` is a method, not a free function** — i wrote `isalnum(s[start])` and got a NameError. it's `s[start].isalnum()`.
3. **short-circuit evaluation order matters** — i wrote `not s[start].isalnum() and start < len(s)`, which still IndexErrors because the index access runs *before* the bounds check. the guard has to come first.
4. **guard against pointer crossing, not just the string ends** — i used `start < len(s)` / `end >= 0` when the right guard is `start < end`, which covers both the bounds and the crossing in one shot.

the moment it clicked was realizing the bounds check in a compound condition is what *protects* the index access after it — so it has to be on the left. once i swapped the order and switched to `start < end`, it passed every edge case including the all-junk string `",.;"`.

---

# Attempt 1

```python
class Solution:
    def isPalindrome(self, s: str) -> bool:
        start = 0
        end = len(s) - 1
        while start < end:
            if s[start] != s[end]:
                return False
            start += 1
            end -= 1
        return True
```

**what i was thinking:** standard converging two-pointer palindrome check.

**what went wrong:** no normalization at all. fails on `"A man, a plan, a canal: Panama"` — dies on the very first compare because `'A' != 'a'`, and non-alphanumeric chars (spaces, commas, colon) were never excluded. correct algorithm shape, but it's solving the wrong problem (raw-string palindrome, not the cleaned version leetcode asks for).

> note: i re-sent this same code with corrected indentation (a copy-paste artifact had flattened it), but the logic was unchanged, so it was still the same bug.

---

# Attempt 2

```python
class Solution:
    def isPalindrome(self, s: str) -> bool:
        start = 0
        end = len(s) - 1
        s.lower()
        while start < end:
            while not isalnum(s[start]):
                start += 1
            while not isalnum(s[end]):
                end -= 1
            if s[start] != s[end]:
                return False
            start += 1
            end -= 1
        return True
```

**what i was thinking:** added the skip-in-loop approach — inner whiles to walk past junk chars, plus a lowercase to fix case.

**what went wrong — three separate bugs:**

- `s.lower()` is a no-op. strings are immutable, so it returns a new string i discarded. `s` is unchanged.
- `isalnum(s[start])` → NameError. `isalnum` is a string method, not a built-in function. needs to be `s[start].isalnum()`.
- the inner whiles have no bounds guard. on an all-junk string like `",.;"`, `start` crawls past the end and `s[start]` throws IndexError.

---

# Attempt 3

```python
class Solution:
    def isPalindrome(self, s: str) -> bool:
        start = 0
        end = len(s) - 1
        s = s.lower()
        while start < end:
            while not s[start].isalnum() and start < len(s):
                start += 1
            while not s[end].isalnum() and end >= 0:
                end -= 1
            if s[start] != s[end]:
                return False
            start += 1
            end -= 1
        return True
```

**what i fixed:** captured the lowercased string (`s = s.lower()`), and called `.isalnum()` correctly as a method.

**what was still wrong:** the bounds guards are in the wrong order. python evaluates `not s[start].isalnum()` *first*, then `start < len(s)`. so on `",.;"`, `s[start]` is accessed before the guard can stop it → still IndexError. also, guarding against `len(s)` / `0` protects the string ends but not the pointers crossing each other.

---

# Attempt 4 (Accepted)

```python
class Solution:
    def isPalindrome(self, s: str) -> bool:
        start = 0
        end = len(s) - 1
        s = s.lower()
        while start < end:
            while start < end and not s[start].isalnum():
                start += 1
            while start < end and not s[end].isalnum():
                end -= 1
            if s[start] != s[end]:
                return False
            start += 1
            end -= 1
        return True
```

**what i fixed:** moved the bounds check to the *left* of the `and` so short-circuit evaluation stops before the index access, and switched the guard to `start < end` so it covers both bounds and crossing. passes all edge cases.

---

# Accepted Solution(s)

```python
class Solution:
    def isPalindrome(self, s: str) -> bool:
        start = 0
        end = len(s) - 1
        s = s.lower()                                  # normalize case (O(n) space — see optimization journey)
        while start < end:
            while start < end and not s[start].isalnum():  # skip junk from the left
                start += 1
            while start < end and not s[end].isalnum():    # skip junk from the right
                end -= 1
            if s[start] != s[end]:                     # business decision: do the pair match?
                return False
            start += 1
            end -= 1
        return True
```

---

# Why It Works

**The invariant:** at every iteration, everything *outside* the `[start, end]` window has already been confirmed to mirror correctly. The two pointers converge inward, and as long as every matched alphanumeric pair is equal, the palindrome property holds. If they ever cross without a mismatch, the whole string mirrored cleanly → it's a palindrome.

The inner whiles exist purely so the pointers only ever land on real, comparable characters — junk is skipped before any comparison happens.

---

# Traversal Logic

How we move through the data — two pointers converging from both ends.

```
"a m a n a p l a n a c a n a l p a n a m a"
 ^                                         ^
 start                                   end

start moves right, end moves left, skipping non-alnum chars,
until they meet or cross in the middle.
```

- `start` advances forward (and skips junk forward)
- `end` retreats backward (and skips junk backward)
- the inner whiles are *also* traversal — they're just fast-forwarding over characters we don't care about

---

# Business Logic

The decision made at each step — separate from how we move.

- at each valid pair (both pointers on alphanumeric chars): **are `s[start]` and `s[end]` equal?**
- mismatch → not a palindrome, return `False` immediately
- survive until pointers cross → return `True`

The *skipping* is traversal. The *comparing* is business. Keep them mentally separate.

---

# Variable Roles

| Variable | Represents | Stores | How It Changes | Why It Exists |
|----------|------------|--------|----------------|---------------|
| `start` | left pointer | int index | `+= 1`, plus skips junk forward | front of the shrinking comparison window |
| `end` | right pointer | int index | `-= 1`, plus skips junk backward | back of the shrinking comparison window |
| `s` | the lowercased input | str | reassigned once via `s = s.lower()` | normalize case so `'A'` and `'a'` compare equal |

---

# Visual Walkthrough

Input: `"0P"` → after `s = s.lower()` → `"0p"`

| Step | start | end | s[start] | s[end] | Action | Result |
|------|-------|-----|----------|--------|--------|--------|
| 1 | 0 | 1 | `'0'` | `'p'` | both alnum, compare | `'0' != 'p'` → return False |

Correct — `"0P"` is not a palindrome.

Input: `",.;"` (all junk, the IndexError trap)

| Step | start | end | inner-left | inner-right | outer check |
|------|-------|-----|-----------|------------|-------------|
| enter outer | 0 | 2 | — | — | `0 < 2` true |
| skip junk left | 0→1→2 | 2 | stops at start==end | — | guard `start < end` halts it at 2 |
| skip junk right | 2 | 2 | — | `start < end` false, never enters | — |
| compare | 2 | 2 | — | — | `s[2] != s[2]`? no, equal |
| step inward | 3 | 1 | — | — | — |
| outer check | 3 | 1 | — | — | `3 < 1` false → return True |

All-junk string is treated as an empty palindrome → `True`. No crash, because every loop is guarded by `start < end` *first*.

---

# Optimization Journey

1. **Brute Force:** build a cleaned string (alnum only, lowercased), then check `cleaned == cleaned[::-1]`. O(n) time, **O(n) space** (two extra strings — the cleaned one and its reverse).
2. **Bottleneck:** the O(n) space. We're allocating whole new strings just to compare ends.
3. **Observation:** we don't need the cleaned string materialized — we can walk the original with two pointers and skip junk in place.
4. **Optimization:** two converging pointers + inner whiles to skip non-alphanumeric.
5. **Final Approach:** the accepted solution. Still has one O(n) cost: `s = s.lower()` allocates a copy. The fully O(1) version (NeetCode sol 2) lowercases *per comparison* instead — `s[start].lower() != s[end].lower()` — and never copies the string.

---

# Pattern Recognition

## Signal Phrases

- "palindrome" → symmetry → compare front to back
- "reads the same forward and backward" → converging pointers
- in-place / O(1) space expectation → two pointers over an extra data structure

## Pattern Used

Two Pointers (converging pair).

## Why This Pattern Fits

- palindrome = symmetric structure, naturally checked from both ends inward
- single pass, no extra data structure needed
- O(1) space achievable (modulo the `.lower()` copy)

---

# Pattern Comparison

| Pattern | Why It Fits | Why It Doesn't | Verdict |
|---------|-------------|----------------|---------|
| Two Pointers | symmetry, converge from ends, O(1) space | — | ✅ correct choice |
| Stack | could push half, pop-compare second half | O(n) space for no benefit | ❌ wasteful |
| String reverse (`s == s[::-1]`) | dead simple, correct | O(n) space, dodges the exercise | ⚠️ works but suboptimal |

---

# Common Mistakes

These are the ones I actually hit on this problem:

- comparing raw chars with no case normalization (`'A' != 'a'`)
- forgetting to exclude non-alphanumeric chars entirely
- assuming `s.lower()` mutates the string (it returns a new one — strings are immutable)
- calling `isalnum(c)` as a function instead of `c.isalnum()` as a method
- putting the bounds check *after* the index access in a compound `while` (short-circuit order)
- guarding inner loops against `len(s)`/`0` instead of `start < end`

---

# Edge Cases

| Case | Input | Expected | How Handled |
|------|-------|----------|-------------|
| empty / all junk | `",.;"` | True | inner whiles guarded by `start < end`, pointers cross, returns True |
| mixed alnum, no junk | `"0P"` | False | direct compare, mismatch caught |
| full sentence w/ punctuation | `"A man, a plan, a canal: Panama"` | True | junk skipped, case normalized via `s.lower()` |
| single char | `"a"` | True | `start < end` false immediately, returns True |

---

# Debugging Lessons

The big one: **in a compound `while` / `if`, the bounds check goes first** so short-circuit evaluation guards the index access that follows it. `while start < end and not s[start].isalnum()` is safe; flipping the order is an IndexError waiting to happen.

Secondary lessons, all python-flavored:
- string methods that "transform" (`.lower()`, `.upper()`, `.strip()`) **return** new strings — capture the result, strings never mutate in place.
- `isalnum` lives *on* the string object — `c.isalnum()`, never `isalnum(c)`.

---

# NeetCode / Official Solution

## Solution 1 — Filter + Reverse

```python
class Solution:
    def isPalindrome(self, s: str) -> bool:
        newStr = ""
        for c in s:
            if c.isalnum():
                newStr += c.lower()
        return newStr == newStr[::-1]
```

Build a cleaned, lowercased string, then check if it equals its own reverse (`[::-1]` is the slice-reverse trick). Dead simple to reason about.

## Solution 2 — Two Pointers (true O(1) space)

```python
class Solution:
    def isPalindrome(self, s: str) -> bool:
        l, r = 0, len(s) - 1
        while l < r:
            while l < r and not self.alphaNum(s[l]):
                l += 1
            while r > l and not self.alphaNum(s[r]):
                r -= 1
            if s[l].lower() != s[r].lower():
                return False
            l, r = l + 1, r - 1
        return True

    def alphaNum(self, c):
        return (ord('A') <= ord(c) <= ord('Z') or
                ord('a') <= ord(c) <= ord('z') or
                ord('0') <= ord(c) <= ord('9'))
```

Same skeleton as mine, with two upgrades: lowercases *per comparison* (`s[l].lower() != s[r].lower()`) instead of copying the whole string, making it genuinely O(1) space; and hand-rolls `alphaNum` with `ord()` range checks instead of `.isalnum()`.

## Comparison to My Solution

| | NeetCode Sol 1 | NeetCode Sol 2 | My Solution |
|---|----------------|----------------|-------------|
| Approach | filter + reverse | two pointers, inline lower | two pointers, `s.lower()` upfront |
| Time | O(n) | O(n) | O(n) |
| Space | O(n) | **O(1)** | O(n) (the `s.lower()` copy) |
| Alnum check | `.isalnum()` | manual `ord()` | `.isalnum()` |
| Bug risk | low (no index juggling) | moderate | moderate |

My logic is identical to Sol 2. The only gap: I copied the whole string to lowercase rather than lowercasing per-comparison. One-line change to reach true O(1).

---

# Deep Understanding

The non-obvious part is **why the nested whiles are still O(n) total and not O(n²).** It looks like a loop inside a loop, which screams O(n²). But `start` and `end` are *monotonic* — they only ever advance toward each other and never reset. Across the entire run, the inner whiles can move the pointers a combined total of at most n steps, because each step permanently shrinks the window. So all the looping put together is bounded by n. The structure lies about its complexity; the pointer behavior tells the truth.

The other subtle line is `ord('A') <= ord(c) <= ord('Z')` in NeetCode's version: it works because uppercase letters, lowercase letters, and digits each occupy *contiguous* ranges in the ASCII table, so a numeric range check on the code point is equivalent to "is this char in that class."

---

# Complexity Analysis

| Metric | Complexity | Reason |
|--------|------------|--------|
| Time | O(n) | each char visited at most once; pointers are monotonic, inner loops don't reset them |
| Space | O(n) | `s = s.lower()` allocates a new n-length string. inline-lowercase version is O(1) |

---

# Related Problems

| # | Problem | Difficulty | Same Pattern? | Notes |
|---|---------|------------|---------------|-------|
| 167 | Two Sum II | Medium | Yes (converging two pointers) | next in section; pointers move based on sum vs target |
| 680 | Valid Palindrome II | Easy | Yes + greedy | allow one deletion; branch on first mismatch |
| 344 | Reverse String | Easy | Yes | pure two-pointer swap, no skipping |
| 11 | Container With Most Water | Medium | Yes | converging pointers, move the smaller side |

---

# Interview Explanation (30 Seconds)

It's a converging two-pointer scan. I start one pointer at each end, skip any non-alphanumeric characters, and compare the pairs case-insensitively, walking inward until they cross. O(n) time, O(1) space if I lowercase per comparison instead of copying the string.

---

# Interview Explanation (2 Minutes)

The signal is "palindrome" — symmetry — which points straight at two pointers converging from both ends. I set `start` at index 0 and `end` at the last index. The invariant is that everything outside the `[start, end]` window has already been verified to mirror correctly.

Inside the loop I first skip non-alphanumeric characters from each side using inner whiles — but crucially each inner while is guarded by `start < end` placed *before* the index access, so short-circuit evaluation prevents an IndexError on all-junk strings. Once both pointers land on real characters, I compare them case-insensitively. A mismatch means it's not a palindrome, so I return False immediately. If the pointers cross without any mismatch, it's a palindrome.

Complexity is O(n) time — the nested whiles look quadratic but the pointers are monotonic and never reset, so total work is linear. Space is O(1) if I lowercase the two compared characters on the fly rather than allocating a lowercased copy of the whole string. The one tradeoff worth mentioning: a filter-then-reverse approach is simpler to write but costs O(n) space.

---

# Common Follow-Up Questions

- **"Make it truly O(1) space."** → drop `s = s.lower()`, compare with `s[start].lower() != s[end].lower()` inline.
- **"Why isn't the nested loop O(n²)?"** → pointers are monotonic, never reset; combined inner-loop movement is bounded by n.
- **"What if you allowed one character deletion?"** → that's LC 680; on first mismatch, branch into two sub-checks (skip left char or skip right char) and accept if either is a palindrome.
- **"Why `ord()` instead of `.isalnum()`?"** → language-portability and showing you understand ASCII ranges; in pure python `.isalnum()` is cleaner. Note `.isalnum()` is broader (unicode), `ord` version is ASCII-only.

---

# Key Takeaways

- two pointers is the go-to for symmetry/palindrome problems — converge from both ends.
- in compound conditions, **bounds check before index access** — short-circuit evaluation is your guard.
- string methods return new strings; they don't mutate. capture the result.
- nested whiles with monotonic pointers are O(n), not O(n²).

---

# Revision Notes

When re-reading in a week/month, the minimum to refresh:
- skeleton: `l, r` converge, skip junk inner-whiles guarded by `l < r` first, compare, step in.
- the one bug to remember: short-circuit ordering (bounds before index).
- the one optimization: inline `.lower()` for O(1) space vs `s.lower()` copy for O(n).

---

# Final Mental Model

Think of two people reading the same word from opposite ends toward the middle, ignoring anything that isn't a letter or number, and checking each letter they meet matches their partner's. As long as every pair matches and they reach the middle, it's a palindrome. The whole trick is they read *in place* — they never write the word out clean on a new page, which is what keeps it memory-free.
