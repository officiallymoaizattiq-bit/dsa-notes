# 20. Valid Parentheses

## Problem

Determine whether a string containing only `()[]{}` is valid.

A string is valid if:
1. Every opening bracket has a matching closing bracket.
2. Brackets close in the correct order.
3. No closing bracket appears without a matching opening bracket.

Example:

```text
Input: "([{}])"
Output: True
```

---

## My Learning Journey

### Initial Intuition

Recognized that the most recently opened bracket must be closed first.

Recognition Signal:

```text
Most recent item must be processed first
=> Stack (LIFO)
```

### Attempt 1

Used a stack and manually matched bracket pairs.

#### Bugs Discovered

1. Incorrect Python `or` syntax.
2. Used `stack.pop` instead of `stack.pop()`.
3. Missed the empty-stack edge case for inputs like:

```text
")"
```

4. Initially had the final stack validation reversed.

#### Key Realizations

- The stack stores unmatched opening brackets.
- A closing bracket with an empty stack is immediately invalid.
- The stack must be empty at the end.

---

## Accepted Solution (NeetCode)

```python
class Solution:
    def isValid(self, s: str) -> bool:
        stack = []
        closeToOpen = {")": "(", "]": "[", "}": "{"}

        for c in s:
            if c in closeToOpen:
                if stack and stack[-1] == closeToOpen[c]:
                    stack.pop()
                else:
                    return False
            else:
                stack.append(c)

        return True if not stack else False
```

---

## Why It Works

### Traversal Logic

Traverse each character once.

```python
for c in s:
```

### Business Logic

- Opening bracket → push to stack.
- Closing bracket → verify it matches the top of the stack.
- Mismatch or empty stack → return `False`.
- Empty stack at the end → return `True`.

---

## Variable Roles

### stack

Stores unmatched opening brackets.

### closeToOpen

Maps closing brackets to the opening bracket they require.

```python
{
    ")": "(",
    "]": "[",
    "}": "{"
}
```

### c

Current character being processed.

---

## Visual Walkthrough

Input:

```text
([{}])
```

| Character | Stack |
|------------|--------|
| ( | ( |
| [ | ([ |
| { | ([{ |
| } | ([ |
| ] | ( |
| ) | empty |

Result:

```text
True
```

---

## Optimization Journey

### Brute Force

Repeatedly remove:

```text
()
[]
{}
```

until no changes remain.

### Bottleneck

Repeated string modifications.

### Observation

Only the most recent unmatched opening bracket matters.

### Optimization

Use a stack.

### Final Approach

Maintain unmatched opening brackets in a stack.

---

## Pattern Recognition

### Pattern

Stack

### Recognition Signals

- Matching pairs
- Nested structures
- Reverse-order processing
- Most recent item matters most

### Why Alternative Patterns Are Weaker

- Counters fail on `([)]`.
- Hash maps alone do not track order.
- Stack naturally enforces ordering.

---

## Common Mistakes

1. Forgetting `()` after `pop`.
2. Not checking if the stack is empty before popping.
3. Reversing the final stack check.
4. Comparing brackets incorrectly.

---

## Edge Cases

```text
""
```

Valid.

```text
"("
```

Invalid.

```text
")"
```

Invalid.

```text
"([)]"
```

Invalid.

```text
"({[]})"
```

Valid.

---

## Complexity Analysis

### My Solution

Time Complexity:

```text
O(n)
```

Space Complexity:

```text
O(n)
```

### NeetCode Solution

Time Complexity:

```text
O(n)
```

Space Complexity:

```text
O(n)
```

---

## Interview Explanation (30 Seconds)

This is a stack problem. Opening brackets are pushed onto the stack. When a closing bracket appears, it must match the most recent unmatched opening bracket at the top of the stack. Any mismatch or empty-stack pop is invalid. The stack must be empty at the end.

---

## Key Takeaways

- Matching-pair problems often imply a stack.
- The stack stores unmatched opening brackets.
- Always check before popping.
- Empty stack at the end means every bracket was matched.

---

## Revision Notes

```text
Stack
Push openings
Match closings
Check before pop
Stack must end empty
```

## Final Mental Model

Think of a stack of plates. Every opening bracket adds a plate. Every closing bracket must remove the top plate. If the wrong plate is needed or the stack is empty, the structure is invalid.
