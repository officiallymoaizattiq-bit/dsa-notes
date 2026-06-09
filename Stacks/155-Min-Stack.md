# 155-Min-Stack

## Problem

Design a stack supporting:

- push()
- pop()
- top()
- getMin()

All operations must run in O(1).

---

## Pattern Recognition

### Recognition Signals

- Stack operations are explicitly required.
- Minimum value queried repeatedly.
- O(1) retrieval requirement.
- Cannot rescan stack every time.

### Pattern

Stack + Auxiliary Min Stack

---

## Solutions

### Solution 1 — Brute Force (Passes Functional Tests, Not Optimal)

```python
class MinStack:
    def __init__(self):
        self.stack = []

    def push(self, val: int) -> None:
        self.stack.append(val)

    def pop(self) -> None:
        self.stack.pop()

    def top(self) -> int:
        return self.stack[-1]

    def getMin(self) -> int:
        return min(self.stack)
```

#### Complexity

| Operation | Time |
|-----------|------|
| push | O(1) |
| pop | O(1) |
| top | O(1) |
| getMin | O(n) |

Space: O(n)

#### Why It Works

Scans the stack whenever getMin() is called.

#### Why It's Not Optimal

Violates the O(1) getMin requirement.

---

### Solution 2 — Single Minimum Variable

```python
class MinStack:
    def __init__(self):
        self.stack = []
        self.min = float("inf")

    def push(self, val):
        self.stack.append(val)
        self.min = min(self.min, val)

    def pop(self):
        self.stack.pop()

    def top(self):
        return self.stack[-1]

    def getMin(self):
        return self.min
```

#### Status

Incorrect

#### Failure Case

```text
push(5)
push(2)
pop()

Expected: 5
Actual: 2
```

#### Why It Fails

Loses previous minimum history.

---

### Solution 3 — My Accepted Solution

```python
class MinStack:

    def __init__(self):
        self.stack=[]
        self.min=[]

    def push(self, val: int) -> None:
        self.stack.append(val)

        if self.min:
            if val <= self.min[-1]:
                self.min.append(val)
        else:
            self.min.append(val)

    def pop(self) -> None:
        if self.stack:
            if self.stack.pop() == self.min[-1]:
                self.min.pop()

    def top(self) -> int:
        return self.stack[-1]

    def getMin(self) -> int:
        return self.min[-1]
```

#### Core Idea

Store minimum history in a second stack.

#### Complexity

| Operation | Time |
|-----------|------|
| push | O(1) |
| pop | O(1) |
| top | O(1) |
| getMin | O(1) |

Space: O(n)

---

### Solution 4 — NeetCode Solution

```python
class MinStack:

    def __init__(self):
        self.stack = []
        self.minStack = []

    def push(self, val: int) -> None:
        self.stack.append(val)

        val = min(val, self.minStack[-1] if self.minStack else val)
        self.minStack.append(val)

    def pop(self) -> None:
        self.stack.pop()
        self.minStack.pop()

    def top(self) -> int:
        return self.stack[-1]

    def getMin(self) -> int:
        return self.minStack[-1]
```

#### Core Idea

Store the minimum value at every stack index.

#### Tradeoff

Uses slightly more memory but simplifies logic.

---

## Traversal Logic

### push()

Append value to stack.

### pop()

Remove top value.

### top()

Read top value.

### getMin()

Read top of min stack.

---

## Business Logic

### My Solution

Push:

```text
If val <= current minimum
store in min stack
```

Pop:

```text
If popped value == current minimum
remove from min stack
```

---

## Variable Roles

### self.stack

Stores actual stack contents.

Example:

```text
[5,2,8,1]
```

### self.min

Stores minimum history.

Example:

```text
[5,2,1]
```

Top element is current minimum.

---

## Visual Trace

```text
push(5)

stack = [5]
min   = [5]
```

```text
push(2)

stack = [5,2]
min   = [5,2]
```

```text
push(8)

stack = [5,2,8]
min   = [5,2]
```

```text
push(1)

stack = [5,2,8,1]
min   = [5,2,1]
```

---

## Edge Cases

### Duplicate Minimums

```text
push(2)
push(2)
pop()
```

Minimum remains 2.

### Single Element

```text
push(5)
pop()
```

### Increasing Sequence

```text
1,2,3,4
```

### Decreasing Sequence

```text
4,3,2,1
```

---

## Bugs I Actually Made

1. Used `stack` instead of `self.stack`.
2. Destroyed stack while searching for minimum.
3. Used hardcoded minimum value.
4. Used a single minimum variable.
5. Used `<` instead of `<=`.
6. Didn't handle duplicate minimums.

---

## Interview Explanation (30 Seconds)

Use two stacks. The main stack stores values, and the min stack stores minimum history. Whenever a new minimum appears, push it into the min stack. When the current minimum is removed, pop it from the min stack as well. This allows getMin() to return the minimum in O(1) time.

---

## Complexity Summary

### Accepted Solution

```text
Push   : O(1)
Pop    : O(1)
Top    : O(1)
GetMin : O(1)

Space  : O(n)
```

---

## Mental Model

The min stack is a history of minimum values.

Whenever a minimum disappears, the previous minimum is already waiting underneath.
