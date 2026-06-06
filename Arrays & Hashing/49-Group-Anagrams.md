# 49. Group Anagrams

## Problem

Given an array of strings `strs`, group the anagrams together. You can return the answer in any order.

An Anagram is a word or phrase formed by rearranging the letters of a different word or phrase, typically using all the original letters exactly once.

---

# My Learning Journey

This problem tested my ability to combine sorting logic with Hash Maps. 

While solving it, I learned:
* You can use the sorted version of a string as a unique "signature" or key to group anagrams together.
* In Python, `sorted("string")` returns a List of characters, not a string. 
* Python Lists are "unhashable" (mutable) and cannot be used as dictionary keys.
* You can solve this by casting the sorted list into a `tuple()`, which is immutable and perfectly valid as a dictionary key.
* Using `.append()` on a dictionary's list modifies it in place and does not require an assignment operator (`=`).
* Returning `list(HashMap.values())` instantly extracts all grouped lists from the dictionary without needing a `for` loop.

---

# Solution 1: Sorting + Hash Map (My Accepted Solution)

## Idea

If two words are anagrams, sorting them alphabetically will always result in the exact same sequence of characters. 

Iterate through the array. For each string, sort it. Use that sorted tuple as a key in a Hash Map. Append the original string to the list associated with that key. Finally, return all the lists from the Hash Map.

## Code

```python
class Solution:
    def groupAnagrams(self, strs: List[str]) -> List[List[str]]:
        HashMap = {}
        for i in range(len(strs)):
            sortedStr = tuple(sorted(strs[i]))
            if sortedStr in HashMap:
                HashMap[sortedStr].append(strs[i]) 
            else:
                HashMap[sortedStr] = [strs[i]]
                
        return list(HashMap.values())
```

## Complexity

Time: $O(m \cdot n \log n)$ 
* Where $m$ is the total number of strings, and $n$ is the length of the longest string.
* We iterate through $m$ strings, and sorting each string takes $n \log n$ time.

Space: $O(m \cdot n)$ 
* Storing all strings in the Hash Map.

## Issues Encountered & Debugged

* **Unhashable Type Error:** Tried to use `sorted(strs[i])` directly as a key, but it evaluates to a List. Wrapped it in `tuple()` to fix it.
* **Syntax error with `.append()`:** I tried doing `HashMap[sortedStr] = .append(...)`. `.append()` operates in-place, so the `=` was causing a crash.
* **Premature Return:** I initially used a `for` loop to return the dictionary values, which caused the function to terminate after returning only the first group. Changed it to return `list(HashMap.values())` all at once.

---

# Solution 2: NeetCode Optimal (Frequency Counting)

## Idea

Sorting each string takes $n \log n$ time. We can bypass sorting entirely by using a **Character Frequency Array** as our "signature" instead.

Since there are only 26 lowercase English letters, we can represent any word as an array of 26 integers. For example, `"eat"` and `"tea"` both have one `a`, one `e`, and one `t`. Their frequency array will look exactly the same. We can convert this array to a tuple and use it as the dictionary key!

## Code

```python
from collections import defaultdict

class Solution:
    def groupAnagrams(self, strs: List[str]) -> List[List[str]]:
        res = defaultdict(list) # mapping charCount to list of Anagrams

        for s in strs:
            count = [0] * 26 # a ... z

            for c in s:
                # ord() gets the ASCII value. ord('a') is 97.
                # 'a' - 'a' = 0 (Index 0)
                # 'z' - 'a' = 25 (Index 25)
                count[ord(c) - ord("a")] += 1

            res[tuple(count)].append(s)

        return res.values()
```

## Complexity

Time: $O(m \cdot n)$
* We loop through $m$ strings, and for each string we just loop through its $n$ characters once to count them. No sorting required!

Space: $O(m \cdot n)$
* For the Hash Map storing all strings.

## How the ASCII Math Works

The line `count[ord(c) - ord("a")] += 1` is a classic math trick.
Computers read letters as ASCII numbers (`a` = 97, `b` = 98, etc.).
To map any letter to an index between 0 and 25, you subtract the value of `a`.
* If `c` is `"e"` (101): `101 - 97 = 4`. The count goes into index 4.
* If `c` is `"z"` (122): `122 - 97 = 25`. The count goes into index 25.

---

# Complexity Comparison

| Solution | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| My Sorting + Hash Map | $O(m \cdot n \log n)$ | $O(m \cdot n)$ | Highly intuitive, very standard accepted answer. |
| NeetCode Frequency Array | $O(m \cdot n)$ | $O(m \cdot n)$ | Optimal. Removes the $n \log n$ sorting bottleneck entirely. |

---

# Key Takeaways

1. **Tuples are magic:** When you need to use a List or an Array as a dictionary key, always cast it to a `tuple()` to make it hashable.
2. **`.append()` works in-place:** Do not use `=` when appending to lists inside a dictionary.
3. **`defaultdict(list)` is a time-saver:** It automatically initializes an empty list for missing keys, saving you from writing standard `if/else` checks.
4. **ASCII Math:** `ord(c) - ord('a')` is the standard way to map lowercase letters to an array of indices `[0 ... 25]`.
5. **Counting beats Sorting:** Just like in Valid Anagram, frequency counting is almost always faster than sorting.
