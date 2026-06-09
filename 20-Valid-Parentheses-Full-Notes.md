# 20. Valid Parentheses

## Problem

Given a string `s` containing only the characters:

```text
(
)
[
]
{
}
```

Determine whether the string is valid.

A string is valid if:

1. Every opening bracket has a matching closing bracket.
2. Brackets close in the correct order.
3. Every closing bracket has a corresponding opening bracket.

### Example

```text
Input: "([{}])"
Output: True
```

---

# My Learning Journey

## Initial Intuition

I identified that the problem involved matching opening and closing brackets.

The key realization was:

```text
The most recently opened bracket must be closed first.
```

This led to recognizing the Stack pattern.

### Pattern Recognition Moment

Recognition Signal:

```text
Most recent item must be processed first
→ Stack (LIFO)
```

### Questions Asked

- What should happen when an opening bracket is encountered?
- What should happen when a closing bracket is encountered?
- What should happen if a closing bracket appears while the stack is empty?
- Why must the stack be empty at the end?

### Visualizations Used

Used stack traces such as:

```text
([{}])

(
([
([{

```

to understand push and pop behavior.

---

# Attempt 1

## Code Discussed

Initial stack implementation.

## What I Was Thinking

- Push opening brackets.
- Pop when a closing bracket appears.
- Compare values.

## Bugs Found

### Syntax Bug

Incorrect Python OR syntax.

### Implementation Bug

Used:

```python
stack.pop
```

instead of:

```python
stack.pop()
```

## Why It Failed

Python was not executing the pop operation.

## What I Learned

Methods must be called using parentheses.

---

# Attempt 2

## What I Was Thinking

Continue using a stack and manually compare brackets.

## Bug Found

Missing handling for closing brackets when the stack is empty.

Example:

```text
)
```

## Why It Failed

A closing bracket cannot be matched if no opening bracket exists.

## What I Learned

Before popping:

```python
if not stack:
```

must be considered.

---

# Attempt 3

## What I Was Thinking

Use explicit comparisons:

```text
) ↔ (
] ↔ [
} ↔ {
```

## Bug Found

Final stack validation was initially reversed.

## Why It Failed

An empty stack means all opening brackets were matched.

## What I Learned

The stack must be empty at the end for the string to be valid.

---

# Accepted Solution 1

```python
class Solution:
    def isValid(self, s: str) -> bool:
        stack=[]
        val=""

        for i in range(len(s)):
            char=s[i:i+1]

            if (char == "(" or char == "[" or char == "{"):
                stack.append(char)

            if (char == ")" or char == "]" or char == "}"):
                if stack:
                    val=stack.pop()

                if (char == ")" and val !="("
                or char == "]" and val != "["
                or char == "}" and val != "{"):
                    return False

        if stack:
            return False

        return True
```

## Why It Works

The stack stores unmatched opening brackets.

Whenever a closing bracket appears, the most recent unmatched opening bracket is checked.

If the pair does not match, the string is invalid.

---

## Traversal Logic

Move through the string one character at a time.

```python
for i in range(len(s)):
```

---

## Business Logic

### Opening Bracket

Push onto stack.

```python
stack.append(char)
```

### Closing Bracket

Pop most recent opening bracket.

```python
val = stack.pop()
```

Verify the pair matches.

---

## Variable Roles

### stack

Stores unmatched opening brackets.

### char

Current character being processed.

### val

Stores the most recently popped opening bracket.

Used for bracket comparison.

---

## Visual Walkthrough

Input:

```text
([{}])
```

| Character | Stack |
|-----------|--------|
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

# Optimization Journey

## Brute Force

Repeatedly remove:

```text
()
[]
{}
```

until no more pairs exist.

## Bottleneck

Repeated string modifications.

## Observation

Only the most recent unmatched opening bracket matters.

## Optimization

Use a stack.

## Final Approach

Maintain unmatched opening brackets and match them as closing brackets appear.

---

# Pattern Recognition

## Pattern

Stack

## Recognition Signals

- Matching pairs
- Nested structures
- Reverse-order processing
- Most recent item matters most

## Why This Pattern Works

Stacks naturally enforce LIFO ordering.

## Why Alternative Patterns Are Weaker

### Counter

Fails on:

```text
([)]
```

Counts match but ordering is wrong.

### Hash Map Alone

Tracks relationships but not ordering.

## Similar Problems

- Min Stack
- Daily Temperatures
- Evaluate Reverse Polish Notation
- Generate Parentheses

---

# Pattern Comparison

## Most Similar Problem

Generate Parentheses

### Similarities

Both involve valid parenthesis relationships.

### Differences

Generate Parentheses constructs combinations while this problem validates them.

---

## Commonly Confused Pattern

Hash Map

A hash map helps identify matching pairs but cannot track nesting order.

---

## Alternative Solution Pattern

Hash Map + Stack

This is the NeetCode approach.

---

## Interview Differences

Interviewers usually expect Stack recognition very quickly.

---

## Signals That Should Trigger This Pattern

- Need reverse-order matching
- Need nested validation
- Most recent item matters most

Signals appearing in this problem:

✅ Reverse-order matching

✅ Nested validation

✅ Most recent item matters most

---

# Common Mistakes

### Mistakes Discovered During Discussion

- Incorrect OR syntax
- Using `stack.pop` instead of `stack.pop()`
- Missing empty-stack handling
- Reversing final stack validation

### Common Interview Mistakes

- Forgetting to check before popping
- Comparing incorrect bracket pairs

### Edge Cases People Miss

```text
)
```

```text
(
```

```text
([)]
```

---

# Edge Cases

## Empty String

Valid because no brackets are unmatched.

## Single Opening Bracket

Invalid because it remains unmatched.

## Single Closing Bracket

Invalid because no opening bracket exists.

## Incorrect Ordering

```text
([)]
```

Invalid despite having equal counts.

## Nested Brackets

```text
({[]})
```

Valid.

---

# Debugging Lessons

## Bugs I Made

- Incorrect OR syntax
- Forgot parentheses on pop
- Missed empty-stack handling
- Reversed final stack validation

## Why I Made Them

Focused on high-level logic before handling all edge cases.

## How To Catch Them Faster Next Time

Manually trace:

```text
)
(
([)]
()
```

before submitting.

---

# NeetCode / Official Solution

```python
class Solution:
    def isValid(self, s: str) -> bool:
        stack = []
        closeToOpen = {")":"(", "]":"[", "}":"{"}

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

# Deep Understanding

## Why It Works

The stack stores unmatched opening brackets.

## Why It Is Optimal

Each character is processed once.

## Important Implementation Choices

Using a stack allows constant-time push and pop operations.

## Data Structure Choices

Stack is the natural choice because of LIFO ordering.

## Key Insight

The most recent unmatched opening bracket is the only bracket that matters when a closing bracket appears.

---

# Complexity Analysis

## My Solution

### Time Complexity

```text
O(n)
```

### Space Complexity

```text
O(n)
```

## NeetCode Solution

### Time Complexity

```text
O(n)
```

### Space Complexity

```text
O(n)
```

---

# Related Problems

## Generate Parentheses

### What Stays The Same

Parenthesis matching concepts.

### What Changes

Generate combinations instead of validating.

---

## Min Stack

### What Stays The Same

Stack operations.

### What Changes

Maintains minimum values.

---

## Daily Temperatures

### What Stays The Same

Stack usage.

### What Changes

Uses monotonic stack logic.

---

# Interview Explanation (30 Seconds)

This is a stack problem. Opening brackets are pushed onto the stack. When a closing bracket appears, it must match the most recent unmatched opening bracket. Any mismatch immediately returns false. The stack must be empty at the end.

---

# Interview Explanation (2 Minutes)

The key observation is that brackets must close in reverse order of opening. That naturally suggests a stack because stacks operate using Last-In First-Out behavior. I traverse the string once. Every opening bracket is pushed onto the stack. When a closing bracket appears, I check whether the most recent unmatched opening bracket matches it. If the stack is empty or the brackets do not match, the string is invalid. After processing all characters, the stack should be empty because every opening bracket should have been matched. This gives O(n) time complexity and O(n) space complexity.

---

# Common Follow-Up Questions

## Why Use A Stack?

Because bracket matching requires reverse-order processing.

## Why Not Use Counters?

Counters cannot detect ordering problems such as:

```text
([)]
```

## Can Space Be Improved?

No. In the worst case all characters may be opening brackets.

---

# Key Takeaways

- Matching-pair problems often indicate a stack.
- Order matters more than counts.
- Always check before popping.
- Empty stack at the end means every bracket was matched.

---

# Revision Notes

```text
Stack
Push openings
Pop closings
Check match
Stack ends empty
```

---

# Final Mental Model

Imagine a stack of plates.

Every opening bracket adds a plate.

Every closing bracket must remove the top plate.

If the wrong plate is needed or no plate exists, the structure is invalid.
