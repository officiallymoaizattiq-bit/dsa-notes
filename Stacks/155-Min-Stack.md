# 155-Min-Stack

## Problem

Design a stack that supports:

- push()
- pop()
- top()
- getMin()

All operations must run in O(1) time.

---

## My Learning Journey

Initially attempted to compute the minimum by scanning the stack.

Realized:

- Scanning is O(n)
- getMin() should not modify the stack
- Storing a single minimum variable fails after popping the minimum
- Need to preserve minimum history

Discovered the auxiliary min-stack pattern.

---

## Attempt 1

Used `stack=[]` inside `__init__`.

### Bugs Found

- Implementation Bug
- Scope Bug

---

## Attempt 2

Tried finding minimum by repeatedly popping values.

### Bugs Found

- Destroyed stack contents
- getMin() became destructive
- Incorrect minimum updates
- Infinite loop possibilities

### Key Realization

Reading data and removing data are different operations.

---

## Attempt 3

Stored a single minimum variable.

### Problem

After removing the minimum value:

`push(5) -> push(2) -> pop()`

Minimum incorrectly remained `2`.

### Key Realization

Need previous minimums.

---

## Attempt 4

Introduced a min stack.

### Problem

Used `<` instead of `<=`.

### Failing Case

`push(5) -> push(2) -> push(2) -> pop()`

Minimum incorrectly became `5`.

### Key Realization

Duplicate minimum values must be preserved.

---

## Accepted Solution(s)

### Solution 1: Auxiliary Min Stack

Maintain:

- `self.stack`
- `self.min`

Whenever `val <= current_min`, push into min stack.

Whenever `popped_value == current_min`, pop min stack.

### NeetCode Solution

Store a minimum value at every stack position.

Example:

Stack = [5,2,8,1]

MinStack = [5,2,2,1]

Pop operations remove from both stacks.

---

## Why It Works

The min stack preserves minimum history.

The top of the min stack always equals the minimum value currently present in the stack.

---

## Traversal Logic

### Push

Append value to stack.

### Pop

Remove top value.

### Top

Read last value.

### GetMin

Read top of min stack.

---

## Business Logic

### Push

Determine whether value should affect minimum tracking.

### Pop

Determine whether removed value was the current minimum.

---

## Variable Roles

### self.stack

Stores actual stack values.

### self.min

Stores minimum history.

---

## Visual Walkthrough

Operations:

push(5)
push(2)
push(8)
push(1)

State:

Stack = [5,2,8,1]

Min = [5,2,1]

After pop():

Stack = [5,2,8]

Min = [5,2]

Current minimum = 2

---

## Optimization Journey

### Brute Force

Scan stack for minimum.

Time: O(n)

### Bottleneck

Repeated scanning.

### Observation

Minimum information can be stored during push.

### Optimization

Maintain a second stack.

### Final Approach

Auxiliary min stack.

---

## Pattern Recognition

### Signals

- Stack operations
- Constant-time minimum retrieval
- Repeated minimum queries

### Pattern

Auxiliary Stack.

---

## Pattern Comparison

### Brute Force

getMin = O(n)

### Min Stack

getMin = O(1)

---

## Common Mistakes

- Forgetting self
- Destroying stack during search
- Using a hardcoded minimum
- Not handling duplicate minimum values
- Using `<` instead of `<=`

---

## Edge Cases

- Duplicate minimums
- Increasing sequence
- Decreasing sequence
- Single element

---

## Debugging Lessons

- State must persist across methods.
- O(1) requirements often require auxiliary data structures.
- Always test duplicate values.

---

## NeetCode / Official Solution

Uses a parallel min stack where each index stores the minimum up to that point.

---

## Deep Understanding

The problem is not about finding the minimum.

The problem is about remembering minimum history efficiently.

---

## Complexity Analysis

- Push: O(1)
- Pop: O(1)
- Top: O(1)
- GetMin: O(1)

Space: O(n)

---

## Related Problems

- 20 Valid Parentheses
- 739 Daily Temperatures
- 84 Largest Rectangle in Histogram

---

## Interview Explanation (30 Seconds)

Use two stacks. One stores values and the other stores minimum information. The top of the min stack always provides the current minimum in O(1).

---

## Interview Explanation (2 Minutes)

The brute-force approach scans the stack whenever getMin() is called, resulting in O(n). To achieve O(1), maintain an auxiliary stack that tracks minimum values and synchronize it during push and pop operations.

---

## Common Follow-Up Questions

1. Why not use one minimum variable?
2. How are duplicate minimums handled?
3. Why is getMin O(1)?
4. Why does the min stack work after pops?

---

## Key Takeaways

- O(1) retrieval usually requires stored state.
- Auxiliary stacks are a common interview pattern.
- Duplicate handling is critical.

---

## Revision Notes

- Stack + Min Stack
- Preserve minimum history
- Duplicate minimums require <=
- getMin should never scan

---

## Final Mental Model

Think of the min stack as a historical record of minimum values. Whenever the current minimum disappears, the previous minimum is immediately available at the top of the min stack.
