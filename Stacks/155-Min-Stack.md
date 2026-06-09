# 155-Min-Stack

## Problem
Design a stack supporting:

- push
- pop
- top
- getMin

All in O(1).

---

## Pattern Recognition

### Signals

- Stack operations
- O(1) retrieval requirement
- Current minimum needed repeatedly

### Pattern

Stack + Auxiliary Stack

---

## Solutions

### Solution 1: Brute Force (Passes but not optimal)

Code:
<full code>

Time:
- push O(1)
- pop O(1)
- top O(1)
- getMin O(n)

Space:
- O(n)

Why It Works:
Scan entire stack whenever getMin is called.

Why It's Bad:
Repeated scans.

---

### Solution 2: Single Minimum Variable

Code:
<full code>

Status:
Incorrect

Failure:
Cannot recover previous minimum after pop.

Example:

push(5)
push(2)
pop()

Expected:
5

Got:
2

---

### Solution 3: Accepted Solution (Mine)

Code:
<full accepted code>

Time:
O(1) all operations

Space:
O(n)

Core Idea:
Min stack stores minimum history.

---

### Solution 4: NeetCode Solution

Code:
<full NeetCode code>

Time:
O(1)

Space:
O(n)

Core Idea:
Store minimum at every index.

Tradeoff:
More memory, simpler logic.

---

## Traversal Logic

push:
Append to stack.

pop:
Remove top.

getMin:
Read top of min stack.

---

## Business Logic

push:
If val <= current minimum:
    update min tracking

pop:
If removed value equals current minimum:
    remove from min stack

---

## Variable Roles

self.stack
- Actual values

self.min
- Minimum history

---

## Visual Trace

push(5)

stack=[5]
min=[5]

push(2)

stack=[5,2]
min=[5,2]

push(8)

stack=[5,2,8]
min=[5,2]

push(1)

stack=[5,2,8,1]
min=[5,2,1]

---

## Edge Cases

Duplicate minimums

push(2)
push(2)
pop()

Minimum remains 2

Single element

Increasing sequence

Decreasing sequence

---

## Common Bugs I Made

1. Forgot self.stack
2. Destroyed stack during getMin
3. Used hardcoded minimum
4. Used single min variable
5. Used < instead of <=

---

## Interview Explanation (30s)

Use two stacks. One stores values and the other stores minimum history. Whenever a new minimum appears, store it in the min stack. When the current minimum is removed, pop it from the min stack. The top of the min stack always provides the minimum value in O(1).

---

## Complexity

Accepted Solution

Push: O(1)
Pop: O(1)
Top: O(1)
GetMin: O(1)

Space: O(n)

---

## Mental Model

The min stack is a history of minimum values.

When a minimum disappears, the previous minimum is already waiting underneath.
