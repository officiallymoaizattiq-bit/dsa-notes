# 242. Valid Anagram

## Problem

Given two strings `s` and `t`, return `true` if `t` is an anagram of `s`, and `false` otherwise.

An anagram is a word formed by rearranging the letters of another word while using all original letters exactly once.

---

# My Learning Journey

This was my second NeetCode problem.

While solving it, I learned:

* Strings are immutable in Python.
* Bubble Sort can be correct but too slow.
* A solution can pass most test cases and still fail due to Time Limit Exceeded (TLE).
* Python's built-in sorting is much faster than Bubble Sort.
* Hash maps can be used to count frequencies.
* Frequency counting leads to the optimal solution.

---

# Solution 1: Bubble Sort (My First Accepted Logic)

## Idea

Convert both strings into lists.

Manually sort both lists using Bubble Sort.

Compare the sorted results.

## Code

```python
class Solution:
    def isAnagram(self, s: str, t: str) -> bool:
        s = list(s)
        t = list(t)

        if len(s) != len(t):
            return False

        for i in range(len(s) - 1):
            for j in range(len(s) - 1):
                if s[j] > s[j + 1]:
                    temp = s[j]
                    s[j] = s[j + 1]
                    s[j + 1] = temp

        for i in range(len(t) - 1):
            for j in range(len(t) - 1):
                if t[j] > t[j + 1]:
                    temp = t[j]
                    t[j] = t[j + 1]
                    t[j + 1] = temp

        for i in range(len(s)):
            if s[i] != t[i]:
                return False

        return True
```

## Result

Passed 26 out of 27 test cases.

Failed the final test case with:

```text
Time Limit Exceeded
```

## Why It Failed

The logic was correct.

Bubble Sort is too slow for large inputs.

## Complexity

Time: O(n²)

Space: O(n)

## Lessons Learned

* Correct solutions can still fail because of performance.
* Bubble Sort is rarely used in real interview solutions.

---

# Solution 2: Built-in sort()

## Idea

Use Python's optimized sorting implementation instead of Bubble Sort.

## Code

```python
class Solution:
    def isAnagram(self, s: str, t: str) -> bool:
        s = list(s)
        t = list(t)

        if len(s) != len(t):
            return False

        s.sort()
        t.sort()

        return s == t
```

## Complexity

Time: O(n log n)

Space: O(n)

## Why It Is Better

Python's sorting algorithm is highly optimized.

This immediately removes the TLE issue.

---

# Solution 3: One-Line Sorting Solution

## Code

```python
class Solution:
    def isAnagram(self, s: str, t: str) -> bool:
        return sorted(s) == sorted(t)
```

## Idea

Sort both strings and compare them directly.

## Complexity

Time: O(n log n)

Space: O(n)

## Notes

This is the shortest sorting-based solution.

---

# Solution 4: Hash Map Frequency Counting (My Optimal Solution)

## Idea

If two strings are anagrams, every character must appear the same number of times in both strings.

Count the frequency of each character using hash maps.

Compare the frequency tables.

## Code

```python
from collections import defaultdict

class Solution:
    def isAnagram(self, s: str, t: str) -> bool:

        if len(s) != len(t):
            return False

        countS = defaultdict(int)
        countT = defaultdict(int)

        for c in s:
            countS[c] += 1

        for c in t:
            countT[c] += 1

        return countS == countT
```

## Example

```text
s = "race"
t = "care"
```

countS:

```text
{
    'r': 1,
    'a': 1,
    'c': 1,
    'e': 1
}
```

countT:

```text
{
    'c': 1,
    'a': 1,
    'r': 1,
    'e': 1
}
```

Both dictionaries are equal.

Return `True`.

## Complexity

Time: O(n)

Space: O(n)

## Why This Is Optimal

Each string is traversed only once.

No sorting is required.

Hash table operations are O(1) on average.

---

# Solution 5: Counter

## Code

```python
from collections import Counter

class Solution:
    def isAnagram(self, s: str, t: str) -> bool:
        return Counter(s) == Counter(t)
```

## Idea

Counter automatically builds frequency tables.

Example:

```python
Counter("aab")
```

produces:

```text
{
    'a': 2,
    'b': 1
}
```

## Complexity

Time: O(n)

Space: O(n)

## Notes

Counter is essentially a specialized frequency-count hash map.

It achieves the same goal as my defaultdict solution with less code.

---

# NeetCode Hash Map Solution

## Code

```python
class Solution:
    def isAnagram(self, s: str, t: str) -> bool:

        if len(s) != len(t):
            return False

        countS, countT = {}, {}

        for i in range(len(s)):
            countS[s[i]] = 1 + countS.get(s[i], 0)
            countT[t[i]] = 1 + countT.get(t[i], 0)

        for c in countS:
            if countS[c] != countT.get(c, 0):
                return False

        return True
```

## Complexity

Time: O(n)

Space: O(n)

## Difference from My Solution

My solution:

```python
return countS == countT
```

NeetCode:

```python
manually compares counts
```

Both are O(n).

Both are accepted.

---

# Complexity Comparison

| Solution                    | Time       | Space |
| --------------------------- | ---------- | ----- |
| Bubble Sort                 | O(n²)      | O(n)  |
| sort() + compare            | O(n log n) | O(n)  |
| sorted() == sorted()        | O(n log n) | O(n)  |
| defaultdict frequency count | O(n)       | O(n)  |
| Counter                     | O(n)       | O(n)  |
| NeetCode Hash Map           | O(n)       | O(n)  |

---

# Key Takeaways

1. Strings are immutable in Python.
2. Bubble Sort works but is inefficient.
3. Time complexity matters.
4. Python's built-in sorting is highly optimized.
5. Hash maps are extremely useful for frequency counting.
6. Counter is a convenience wrapper around the frequency-count idea.
7. Many string interview problems can be solved by counting occurrences.
8. This was my first problem where I improved a solution from O(n²) to O(n).
9. Understanding the hash map approach is more valuable than memorizing Counter.
