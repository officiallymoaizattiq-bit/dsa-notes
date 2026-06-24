# 76. Minimum Window Substring

## Problem
Given strings `s` and `t`, return the shortest substring of `s` that contains every character of `t` including duplicates. If no such window exists, return `""`. Junk characters inside the window are allowed — you just need to *cover* `t`.

Example: `s = "ADOBECODEBANC"`, `t = "ABC"` → `"BANC"` (contains A, B, C and nothing shorter does).

---

# My Learning Journey

Came in with the right skeleton instinct — two dicts (`dictt` = what's needed, `dictcurr` = what the window holds), expand `r`, shrink `l`. The pattern recognition was there immediately: variable-size sliding window. Where it got hard was the *validity check*.

First instinct for the shrink trigger was `while dictcurr == dictt` — full dict equality. That was the first real wall: it's both too strict (junk chars in the window break equality forever) and conceptually the wrong tool. Got walked to the `have`/`need` two-integer mechanism, which is the actual lesson of this problem.

From there it was a chain of boundary-condition bugs, each one a real conceptual gap:
- counting `have` on *every* needed char instead of only when a requirement gets *exactly* met (edge-triggered vs level-triggered)
- mutating the wrong dict (`dictt` instead of `dictcurr`)
- KeyError from dropping the `in dictt` guard
- measuring the window *after* shrinking instead of before
- decrementing `have` unconditionally on shrink
- saving a *length* instead of *boundaries* (can't return a number)
- off-by-one on the final slice
- a sentinel (`len(s)`) that collides with a real window length

Got to Accepted, but needed the full hint chain. The `have`/`need` edge-trigger is the part that clicked hardest and is worth re-deriving cold.

---

# Attempt 1 — `dictcurr == dictt` shrink

```python
while dictcurr == dictt:
    dictcurr[s[l]] -= 1
    l += 1
```

Thinking: "shrink while the window equals what I need." Felt clean, never fires.

| Bug | Type | Explanation |
|-----|------|-------------|
| `dictcurr == dictt` as validity check | Logic | Window collects chars not in `t` (D, O, E...), so `dictcurr` gets extra keys and higher counts. `==` demands an exact match, which basically never happens. The real condition is "window *contains* t" — for every char in t, `dictcurr` has *at least* as many. Also it's a full dict walk every iteration when a single integer compare would do. |

Trace that exposed it (`s="ADOBECODEBANC"`, `t="ABC"`, `dictt={A:1,B:1,C:1}`):

| r | char | dictcurr | == dictt? |
|---|------|----------|-----------|
| 0 | A | {A:1} | no |
| 1 | D | {A:1,D:1} | no |
| 2 | O | {A:1,D:1,O:1} | no |
| 3 | B | {A:1,D:1,O:1,B:1} | no — junk keys forever |

---

# Attempt 2 — have/need, but over-counting + wrong dict

```python
if s[r] in dictt:
    have += 1
    dictt[s[r]] -= 1
```

Thinking: introduced `have`/`need`, but bumped `have` for any needed char and decremented the target dict.

| Bug | Type | Explanation |
|-----|------|-------------|
| `have += 1` on every needed char | Logic | `have` should count *distinct requirements satisfied*, not occurrences. With `t="AABC"` (needs A:2), seeing two A's pushes `have` to 2 when only **one** requirement (the A requirement) is met. `have` overshoots `need`. |
| `dictt[s[r]] -= 1` | Implementation | `dictt` is the fixed target — never mutate it. Counts go into `dictcurr`. |
| `need = len(dictt)` placed before the build loop | Implementation/Ordering | `dictt` is empty there, so `need = 0`. Must come *after* building `dictt`. |

---

# Attempt 3 — KeyError + measure-after-shrink

```python
if dictcurr[s[r]] == dictt[s[r]]:
    have += 1
while have == need:
    dictcurr[s[l]] -= 1
    l += 1
    curlen = r - l + 1
    shortest = min(shortest, curlen)
```

Thinking: edge-trigger the count on `==`, shrink and measure.

| Bug | Type | Explanation |
|-----|------|-------------|
| `dictt[s[r]]` without `in dictt` guard | Edge Case | When `s[r]` isn't in `t` (a `Z`, a `D`), `dictt[s[r]]` → **KeyError**, crash. Need `if s[r] in dictt and dictcurr[s[r]] == dictt[s[r]]`. |
| Measure after `l += 1` | Logic | The valid window exists at the *top* of the while, before chopping. Measuring after shrinking records a window that was just invalidated. |
| `have` never decremented | Logic | Once `have == need`, it stays there → `while` never exits → infinite loop or `l` runs off the end and KeyErrors on `dictcurr[s[l]]`. |
| `shortest = min(curlen)` tracks length only | Logic | Problem returns a substring, not a number. Need to record boundaries. |

---

# Attempt 4 — guard + boundaries, but off-by-one + bad sentinel

```python
shortest = len(s)
...
while have == need:
    if (r - l + 1) < shortest:
        shortest = r - l + 1
        resL, resR = l, r
    dictcurr[s[l]] -= 1
    if s[l] in dictt and dictcurr[s[l]] < dictt[s[l]]:
        have -= 1
    l += 1
return s[resL:resR]
```

Thinking: measure-before-shrink fixed, guarded `have -= 1` added, saving boundaries now.

| Bug | Type | Explanation |
|-----|------|-------------|
| `s[resL:resR]` | Edge Case / Off-by-one | Python slicing is right-exclusive. `resR` is the index of the last char *in* the window, so this drops it. Window "BANC" → "BAN". Need `s[resL:resR+1]`. |
| `shortest = len(s)` as "not found" | Logic / Edge Case | `len(s)` is an achievable window length, so "found nothing" is indistinguishable from "found a window of length len(s)." Worse, with `s="a"`, `t="aa"` the while never runs, `resL`/`resR` never get assigned → **NameError**. Use `float("inf")`. |

Breaking input: `s="a"`, `t="aa"` → never enters while → NameError on undefined `resL`.

---

# Accepted Solution

```python
class Solution:
    def minWindow(self, s: str, t: str) -> str:
        dictt = {}
        have = 0
        dictcurr = {}
        shortest = float("inf")
        for i in range(len(t)):
            dictt[t[i]] = 1 + dictt.get(t[i], 0)
        need = len(dictt)
        l = 0
        for r in range(len(s)):
            dictcurr[s[r]] = 1 + dictcurr.get(s[r], 0)
            if s[r] in dictt and dictcurr[s[r]] == dictt[s[r]]:
                have += 1
            while have == need:
                if (r - l + 1) < shortest:
                    shortest = r - l + 1
                    resL, resR = l, r
                dictcurr[s[l]] -= 1
                if s[l] in dictt and dictcurr[s[l]] < dictt[s[l]]:
                    have -= 1
                l += 1
        if shortest != float("inf"):
            return s[resL:resR+1]
        else:
            return ""
```

Result: Accepted. (Indentation in the chat pastes was a copy artifact — the real structure has both `for` loops at method level, conditionals nested in their loops.)

---

# Why It Works
**Invariant:** `have == need` is true exactly when every *distinct* character requirement of `t` is currently met by the window.

**Key operation:** maintain that invariant with one integer instead of recomparing two dicts. `have` only changes at *boundary crossings* — `+1` the instant a count reaches its target on expand (`==`), `-1` the instant a count falls below its target on shrink (`<`). Edge-triggered, fires once per crossing.

---

# Traversal Logic
Two one-way pointers, both only ever move forward.

- `r` (outer for-loop) marches left→right across `s`, expanding the window and adding chars to `dictcurr`.
- `l` advances only inside the while, contracting the window from the left while it stays valid.

```
s = A D O B E C O D E B A N C
    l→ → → → → → → → → →
              r→ → → → → → →
window = s[l..r] inclusive
```

`l` never resets, never backtracks. Across the whole run it advances at most `len(s)` times total.

---

# Business Logic
Per outer step (`r`):

1. Add `s[r]` to `dictcurr`.
2. If `s[r]` is required AND its window count just *reached* the required count → a requirement is newly satisfied → `have += 1`.
3. While the window is fully valid (`have == need`):
   1. If current window is shorter than best so far, save its length and boundaries.
   2. Pop `s[l]` from `dictcurr`.
   3. If `s[l]` was required AND its count just *fell below* the requirement → a requirement is newly broken → `have -= 1`.
   4. Advance `l`.
4. At the end, slice out the saved window or return `""`.

---

# Variable Roles
| Variable | Represents | Stores | How It Changes | Why It Exists |
|----------|------------|--------|----------------|---------------|
| `dictt` | The target — what `t` requires | char → required count | Built once, never mutated | Fixed reference for "what does a valid window need" |
| `dictcurr` | The live window contents | char → current count in window | `+1` on expand at `r`, `-1` on shrink at `l` | Tracks what the window actually holds right now |
| `need` | Number of distinct requirements | `len(dictt)` | Set once after build | The target value for `have` |
| `have` | Requirements currently satisfied | int | `+1` when a count hits target, `-1` when one drops below | Lets validity be a single `==` compare instead of a dict walk |
| `l` | Left edge of window | index into `s` | `+= 1` inside while | Shrinks the window to find the minimum |
| `r` | Right edge of window | index into `s` | for-loop | Expands the window |
| `shortest` | Best window length found | int / `inf` | Set when a smaller valid window appears | Comparison + "not found" sentinel |
| `resL, resR` | Boundaries of best window | indices into `s` | Set alongside `shortest` | So the substring can be sliced out at the end |

---

# Visual Walkthrough
`s = "ADOBECODEBANC"`, `t = "ABC"`, `dictt = {A:1, B:1, C:1}`, `need = 3`.

| r | char | dictcurr (relevant) | have | valid? | l after shrink | best window |
|---|------|---------------------|------|--------|----------------|-------------|
| 0 | A | A:1 | 1 | no | 0 | — |
| 1 | D | A:1 | 1 | no | 0 | — |
| 2 | O | A:1 | 1 | no | 0 | — |
| 3 | B | A:1,B:1 | 2 | no | 0 | — |
| 4 | E | A:1,B:1 | 2 | no | 0 | — |
| 5 | C | A:1,B:1,C:1 | 3 | YES | shrink → l moves to 1 (drop A, have→2) | "ADOBEC" (len 6) |
| ... | | window slides | | | | |
| 12 | C | ...A:1,B:1,C:1 | 3 | YES | shrink past D,E,B... lands tight | "BANC" (len 4) |

The shrink at r=5 records "ADOBEC", then drops the leading A which breaks the A requirement (`have` → 2) and kicks out of the while. Window keeps sliding until "BANC" wins.

---

# Optimization Journey
1. **Brute force:** check every substring of `s`, test if it covers `t`. O(s² · t) or so. Way too slow.
2. **Bottleneck:** re-scanning every candidate window from scratch, and re-validating coverage by walking the whole requirement set each time.
3. **Observation:** a window only gains/loses validity at its *edges*. Adding one char can satisfy at most one requirement, removing one char can break at most one. No need to recheck everything.
4. **Optimization:** sliding window + a single `have`/`need` counter that's edge-triggered on count-crossings, so validity is an O(1) compare.
5. **Final:** expand `r` always, shrink `l` whenever valid, record the min. The one-way `l` makes the nested loop amortized linear.

---

# Pattern Recognition
## Signal Phrases
- "shortest / minimum substring containing..." → variable-size sliding window (shrink to minimize)
- "contains all characters of t (including duplicates)" → count-based coverage → hashmap + `have`/`need`
- "substring" (contiguous) → window, not subsequence

## Pattern Used
Sliding Window (variable size) + Hash Map.

## Why It Fits
- Answer is a contiguous range → window.
- "Minimum valid window" → expand to become valid, shrink to minimize.
- Duplicate-sensitive coverage → counts, not a set.

---

# Pattern Comparison
| Pattern | Why It Fits | Why It Doesn't | Verdict |
|---------|-------------|----------------|---------|
| Sliding Window | Contiguous min window with a validity condition | — | ✅ correct |
| Two Pointers (plain) | `l`/`r` move inward/forward | No sorted structure or pair-target; needs the count machinery a bare two-pointer lacks | Subset of sliding window here |
| Hash Set | Tracks "seen" chars | Can't handle duplicate counts (t may need 2 of a char) | ❌ loses duplicate info |
| Hash Map | Needed for the counts | Not standalone — it's the bookkeeping inside the window | ✅ as a component |

---

# Common Mistakes
- Using `==` between the two dicts for validity (junk chars break it forever). *(hit this)*
- Counting `have` on every needed-char occurrence instead of only at the exact `==` crossing. *(hit this)*
- Mutating the target dict instead of the window dict. *(hit this)*
- Forgetting the `in dictt` guard → KeyError on chars not in `t`. *(hit this)*
- Measuring the window after shrinking instead of before. *(hit this)*
- Decrementing `have` unconditionally on shrink. *(hit this)*

---

# Edge Cases
| Case | Input | Expected | How Handled |
|------|-------|----------|-------------|
| `t` longer than `s` | `s="a"`, `t="aa"` | `""` | `have` never reaches `need`, `shortest` stays `inf` → return `""` |
| No valid window | `s="abc"`, `t="xyz"` | `""` | same — sentinel catches it |
| Window has junk chars | `s="ADOBEC"`, `t="ABC"` | `"ADOBEC"` then tighter | `in dictt` guard ignores junk, coverage still met |
| Duplicate requirements | `t="AABC"` | needs 2 A's | counts in `dictt`, edge-trigger fires only when count hits 2 |
| `t` empty (NeetCode guards this) | `t=""` | `""` | NeetCode adds `if t == "": return ""` up front |

---

# Debugging Lessons
- **Edge-triggered counting is the whole problem.** `have += 1` only at the `==` crossing, `have -= 1` only at the `<` crossing. Level-triggered ("is this char needed") silently corrupts the counter. This was the single biggest gap.
- **Measure before you mutate.** The valid window is the one at the top of the while, before any shrink touches it.
- **Recurring-bug repeat — off-by-one on a right-exclusive slice.** `s[resL:resR]` dropped the last char. Python slices exclude the right bound; an inclusive end index needs `+1`.
- **Recurring-bug repeat — sentinel colliding with a real value.** `len(s)` can't mean "not found" because it's an achievable length. Reach for `float("inf")` or `-1`.

---

# NeetCode / Official Solution

```python
class Solution:
    def minWindow(self, s: str, t: str) -> str:
        if t == "": return ""

        countT, window = {}, {}
        for c in t:
            countT[c] = 1 + countT.get(c, 0)

        have, need = 0, len(countT)
        res, resLen = [-1, -1], float("infinity")
        l = 0
        for r in range(len(s)):
            c = s[r]
            window[c] = 1 + window.get(c, 0)

            if c in countT and window[c] == countT[c]:
                have += 1

            while have == need:
                # update our result
                if (r - l + 1) < resLen:
                    res = [l, r]
                    resLen = (r - l + 1)
                # pop from the left of our window
                window[s[l]] -= 1
                if s[l] in countT and window[s[l]] < countT[s[l]]:
                    have -= 1
                l += 1
        l, r = res
        return s[l:r+1] if resLen != float("infinity") else ""
```

## Comparison to My Solution
| Aspect | Mine | NeetCode |
|--------|------|----------|
| Need/have mechanism | identical | identical |
| Expand condition | `s[r] in dictt and dictcurr[s[r]] == dictt[s[r]]` | same (`c in countT and window[c] == countT[c]`) |
| Shrink condition | `s[l] in dictt and dictcurr[s[l]] < dictt[s[l]]` | same |
| Result storage | `resL, resR` two ints | `res = [l, r]` single list, unpacked at end |
| `c = s[r]` alias | repeats `s[r]` inline | caches `c = s[r]` (minor readability) |
| Empty-`t` guard | none (problem constraints make `t` non-empty, but it's a free safety net) | explicit `if t == "": return ""` |
| Sentinel | `float("inf")` | `float("infinity")` (same thing) |

Verdict: functionally the same algorithm. The only real deltas are cosmetic — NeetCode caches `c`, packs the result in a list, and adds an empty-`t` guard worth copying.

---

# Deep Understanding
The load-bearing line is `window[c] == countT[c]` (note: `==`, not `>=`). It looks like it should be `>=` since "covered" means "at least enough." But `>=` would re-fire `have += 1` on every extra occurrence past the requirement, blowing past `need`. The exact `==` makes it edge-triggered: it's true at the precise moment the count *crosses* from insufficient to sufficient, and only then. Mirror logic on shrink: `< countT[s[l]]` fires only at the moment it crosses back. Those two strict comparisons are what keep `have` an honest count of satisfied requirements.

---

# Complexity Analysis
| Metric | Complexity | Reason |
|--------|------------|--------|
| Time | O(s + t) | Build `dictt` in O(t). Main loop: `r` walks `s` once; `l` only moves forward and never resets, so the nested while contributes O(s) *total* across the whole run, not O(s) per step. Amortized linear. |
| Space | O(s + t) | `dictt` bounded by distinct chars in `t`, `dictcurr` by distinct chars in `s`. (Constant if you assume a fixed alphabet — but state O(s+t) in interview.) |

**Amortized callout:** the while nested in the for *looks* O(n²). It isn't. `l` is a one-way pointer that advances at most `n` times over the entire execution. A non-resetting forward pointer = O(n) total, regardless of how it's nested. This is the reusable insight, not a footnote.

---

# Related Problems
| # | Problem | Difficulty | Same Pattern? | Notes |
|---|---------|------------|---------------|-------|
| 3 | Longest Substring Without Repeating Characters | Medium | Yes (variable window) | Shrink on *invalid* instead of to-minimize; maximize not minimize |
| 567 | Permutation in String | Medium | Yes + counts | Fixed-size window, same have/need idea |
| 438 | Find All Anagrams in a String | Medium | Yes + counts | Fixed window, collect all matches |
| 209 | Minimum Size Subarray Sum | Medium | Yes (min window) | Numeric sum condition instead of char coverage |

---

# Interview Explanation (30s)
Variable-size sliding window with two hashmaps — one for what `t` needs, one for the current window. I expand right always, and whenever the window covers `t`, I shrink from the left to minimize, recording the smallest valid window. Validity is tracked with a single `have == need` counter, so it's O(s + t) time.

# Interview Explanation (2 min)
Signal: "minimum substring containing all of t, with duplicates" → variable sliding window plus counts (a set won't do because duplicates matter). I build `countT` from `t`, then track `have` (distinct requirements currently met) against `need` (`len(countT)`). Expanding right, I increment the window count for that char and bump `have` only when that char's window count exactly *reaches* its required count — the edge-trigger, `==` not `>=`, so it fires once per requirement. While `have == need` the window is valid, so I record it if it's the new shortest, then pop from the left, decrementing `have` only when a char's count falls *below* its requirement, which is what eventually breaks validity and ends the shrink. Invariant: `have == need` ⟺ window covers `t`. Complexity is O(s + t) — `l` never backtracks so the nested loop is amortized linear — and O(s + t) space. Tradeoff: the counter bookkeeping is more fiddly than a dict-equality check, but it turns per-step validation from O(alphabet) into O(1).

---

# Common Follow-Up Questions
- **What if the alphabet is huge / Unicode?** Hashmaps already handle it; space is O(distinct chars), no fixed-256 assumption needed.
- **What if you needed *all* minimum windows, not just one?** Track all windows of length `shortest` instead of overwriting; collect on ties.
- **Why `==` on expand and `<` on shrink, not `>=`/`<=`?** Edge-triggering — they fire exactly at the crossing so `have` counts requirements, not occurrences.
- **Can you do it without two hashmaps?** One map of `countT` and a running deficit counter also works; the two-map form is just clearest.

---

# Key Takeaways
- Coverage-with-duplicates → hashmap counts, never a set.
- Track window validity with an edge-triggered integer (`have`/`need`), not repeated structure comparison.
- `==` / `<` at the crossings is what makes the counter honest — `>=` / `<=` corrupts it.
- A forward-only pointer nested in a loop is amortized O(n), not O(n²).
- Return substrings via saved *boundaries*, not a length. Slice is right-exclusive → `+1`.
- "Not found" needs a sentinel no real answer can equal (`inf`, `-1`).

---

# Revision Notes
**1 week:** Re-state the invariant (`have == need` ⟺ window covers t) and re-trace the breaking input `s="ADOBECODEBANC"`, `t="ABC"` to confirm `have` only moves at `==`/`<` crossings.
**1 month:** Rebuild the one-line skeleton (expand always → bump have at `==` → while valid: record, pop left, drop have at `<`) and state why it's O(s+t) (l never backtracks).

---

# Final Mental Model
Think of `t` as a shopping list with quantities. You walk a sliding window down `s` like a cart rolling through aisles, tossing in whatever you pass. `have` is "how many items on my list are fully checked off." The moment all items are checked, you've got a valid cart — so you start tossing stuff out the *back* of the cart to make it as small as possible, stopping the instant you drop something that un-checks a list item. You remember the smallest full cart you ever had. Junk you grabbed that's not on the list doesn't matter — you only care that the list is covered.
