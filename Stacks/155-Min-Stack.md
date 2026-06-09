# 155-Min-Stack

## 1. Problem

Design a stack that supports:

- push(val)
- pop()
- top()
- getMin()

All operations must run in O(1) time.

---

## 2. My Learning Journey

I started by treating the problem as a "find the minimum" problem.

My first instinct was to scan the stack inside getMin().

That led to multiple issues:

1. The stack variable was not stored correctly.
2. getMin() was destroying the stack while searching.
3. A single minimum variable could not recover after pops.
4. Duplicate minimum values broke the logic.
5. Eventually I realized the problem is not finding the minimum—it is preserving minimum history.

The major breakthrough was discovering the auxiliary min stack pattern.

---

## 3. Attempt 1

### Idea

Store values in a stack and scan the stack when getMin() is called.

### Problems Found

- Used `stack=[]` instead of `self.stack=[]`.
- Methods could not access the same stack.
- getMin() popped values while searching.
- Minimum variable was never updated correctly.
- Could enter infinite loops.

### Lesson

Object state must be stored using instance variables and getMin() should never destroy data.

---

## 4. Attempt 2

### Idea

Fix stack access using `self.stack`.

### Problems Found

- Called `self.pop(stack)`.
- pop() returned None.
- Compared None against numbers.
- Popped multiple times while searching.

### Lesson

Understand exactly what functions return and how many times they are being called.

---

## 5. Additional Attempts

### Attempt 3: Single Minimum Variable

Idea:

```python
self.min
```

Store one minimum value.

#### Failure Case

```text
push(5)
push(2)
pop()
```

Expected minimum:

```text
5
```

Stored minimum:

```text
2
```

#### Lesson

One minimum variable loses previous minimum history.

---

### Attempt 4: Min Stack

Created:

```python
self.min
```

to store minimum values.

#### Failure Case

```text
push(5)
push(2)
push(2)
pop()
```

Used:

```python
<
```

instead of:

```python
<=
```

#### Lesson

Duplicate minimum values must be tracked.

---

## 6. Accepted Solution(s)

### My Accepted Solution

Maintain:

```python
self.stack
self.min
```

Push into min stack whenever:

```python
val <= current_min
```

Pop from min stack whenever:

```python
popped_value == current_min
```

### NeetCode Solution

Stores a minimum value for every position.

Example:

```text
Stack    = [5,2,8,1]
MinStack = [5,2,2,1]
```

Lengths always match.

---

## 7. Why It Works

Invariant:

The top of the min stack is always the minimum value currently present in the main stack.

Whenever a minimum enters the stack, it is recorded.

Whenever that minimum leaves the stack, it is removed from the min stack.

---

## 8. Traversal Logic

### push()

Append value to top of stack.

### pop()

Remove value from top of stack.

### top()

Read last element.

### getMin()

Read top of min stack.

---

## 9. Business Logic

### push()

Determine whether the new value affects minimum tracking.

### pop()

Determine whether the removed value was the current minimum.

### getMin()

Return current minimum instantly.

---

## 10. Variable Roles

### self.stack

Stores actual stack contents.

Example:

```text
[5,2,8]
```

### self.min

Stores minimum history.

Example:

```text
[5,2]
```

Top represents current minimum.

---

## 11. Visual Walkthrough

Operations:

```text
push(5)
push(2)
push(8)
push(1)
```

State:

| Main Stack | Min Stack |
|------------|------------|
| [5] | [5] |
| [5,2] | [5,2] |
| [5,2,8] | [5,2] |
| [5,2,8,1] | [5,2,1] |

After:

```text
pop()
```

State:

```text
Main = [5,2,8]
Min  = [5,2]
```

Current minimum:

```text
2
```

---

## 12. Optimization Journey

### Brute Force

Scan stack during getMin().

Time:

```text
O(n)
```

### Bottleneck

Repeated scanning.

### Observation

Minimum information can be stored during push.

### Optimization

Use an auxiliary stack.

### Final Approach

Maintain minimum history in O(1).

---

## 13. Pattern Recognition

### Signals

- Stack operations
- O(1) minimum retrieval
- Repeated minimum queries

### Pattern

Stack + Auxiliary Min Stack

---

## 14. Pattern Comparison

### Brute Force

- getMin(): O(n)

### My Solution

- Stores only minimum values.
- Slightly less memory usage.

### NeetCode Solution

- Stores minimum at every position.
- Simpler pop logic.

---

## 15. Common Mistakes

Mistakes that actually occurred:

- Forgetting self.
- Destroying stack while searching.
- Using a hardcoded minimum.
- Using one minimum variable.
- Not handling duplicate minimums.
- Using `<` instead of `<=`.

---

## 16. Edge Cases

### Duplicate Minimums

```text
2,2
```

### Single Element

```text
[5]
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

## 17. Debugging Lessons

- Trace state after every operation.
- Test duplicate values.
- Think about what happens after pop().
- O(1) requirements usually require stored state.

---

## 18. NeetCode / Official Solution

NeetCode keeps both stacks the same length.

For every index:

```text
minStack[i]
```

stores the minimum value seen up to that point.

Tradeoff:

- More memory
- Simpler implementation

---

## 19. Deep Understanding

This problem is not about computing a minimum.

It is about preserving historical minimum information so that previous minimums can be recovered after pops.

---

## 20. Complexity Analysis

| Operation | Time |
|-----------|------|
| push | O(1) |
| pop | O(1) |
| top | O(1) |
| getMin | O(1) |

Space:

```text
O(n)
```

---

## 21. Related Problems

- 20 Valid Parentheses
- 739 Daily Temperatures
- 84 Largest Rectangle in Histogram
- Frequency Stack

---

## 22. Interview Explanation (30 Seconds)

Use two stacks. One stores values and the other stores minimum information. When a new minimum is pushed, record it in the min stack. When the current minimum is removed, remove it from the min stack too. The top of the min stack always gives the minimum value in O(1).

---

## 23. Interview Explanation (2 Minutes)

A brute-force solution scans the stack whenever getMin() is called, resulting in O(n) time. To achieve O(1), maintain an auxiliary stack that tracks minimum values. During push, update minimum history. During pop, synchronize the min stack whenever the current minimum is removed. This preserves constant-time access to the current minimum while correctly handling duplicate minimum values.

---

## 24. Common Follow-Up Questions

1. Why not use one minimum variable?
2. How are duplicate minimums handled?
3. Why is getMin() O(1)?
4. Why does the min stack work after pops?
5. What tradeoff does the NeetCode solution make?

---

## 25. Key Takeaways

- O(1) queries often require extra storage.
- Preserve history when information may be needed later.
- Always test duplicate values.
- Stack problems often rely on auxiliary stacks.

---

## 26. Revision Notes

- Pattern: Stack + Min Stack
- getMin must be O(1)
- Preserve minimum history
- Use <= for duplicate minimums
- Top of min stack = current minimum

---

## 27. Final Mental Model

Think of the min stack as a backup history of minimum values. Whenever a new minimum appears, record it. Whenever that minimum disappears, remove it. The top of the min stack always tells you the current minimum without searching.
