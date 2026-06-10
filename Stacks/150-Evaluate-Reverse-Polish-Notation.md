# 150-Evaluate-Reverse-Polish-Notation

# Problem

Evaluate an arithmetic expression in Reverse Polish Notation (postfix notation).

Supported operators:

- `+`
- `-`
- `*`
- `/`

Division truncates toward zero.

---

# Pattern Recognition

## Recognition Signals

- Reverse Polish Notation (RPN)
- Postfix expression
- Evaluate expression
- Operators consume previous values
- Most recent operands are used first

## Pattern Used

- Stack

## Why This Pattern Fits

- Operators need the most recently seen operands.
- Stack provides LIFO access.
- Expressions can be evaluated incrementally.

---

# Solutions

## Solution 1 — My Accepted Solution

### Status

Accepted

### Full Code

```python
class Solution:
    def evalRPN(self, tokens: List[str]) -> int:
        stack=[]
        for s in tokens:
            if s == "+":
                val1=stack.pop()
                val2=stack.pop()
                val2 += val1
                stack.append(val2)
            elif s == "-":
                val1=stack.pop()
                val2=stack.pop()
                val2 -= val1
                stack.append(val2)
            elif s == "*":
                val1=stack.pop()
                val2=stack.pop()
                val2 *= val1
                stack.append(val2)
            elif s == "/":
                val1=stack.pop()
                val2=stack.pop()
                val2 = int(val2 / val1)
                stack.append(val2)
            else:
                stack.append(int(s))
        return stack[-1]
```

### Core Idea

- Push numbers.
- For operators:
  - Pop right operand.
  - Pop left operand.
  - Compute result.
  - Push result.

### Time Complexity

O(n)

### Space Complexity

O(n)

### Tradeoffs

Pros:
- Explicit operand handling
- Easy to debug
- Interview friendly

Cons:
- More verbose than compact solutions

---

## Solution 2 — NeetCode Style Solution

### Status

Accepted

### Full Code

```python
class Solution:
    def evalRPN(self, tokens: List[str]) -> int:
        stack = []

        for c in tokens:
            if c == "+":
                stack.append(stack.pop() + stack.pop())

            elif c == "-":
                a, b = stack.pop(), stack.pop()
                stack.append(b - a)

            elif c == "*":
                stack.append(stack.pop() * stack.pop())

            elif c == "/":
                a, b = stack.pop(), stack.pop()
                stack.append(int(b / a))

            else:
                stack.append(int(c))

        return stack[0]
```

### Core Idea

Same stack pattern with more compact code.

### Time Complexity

O(n)

### Space Complexity

O(n)

### Tradeoffs

Pros:
- Shorter implementation

Cons:
- Operand order less explicit

---

# Traversal Logic

How data moves.

```text
Read token
↓
Number?
↓
Push onto stack

Operator?
↓
Pop right operand
↓
Pop left operand
↓
Compute result
↓
Push result
```

---

# Business Logic

What decisions are made.

Invariant:

```text
The stack contains values of fully evaluated subexpressions.
```

For every operator:

```text
val1 = first pop  = right operand
val2 = second pop = left operand

result = val2 OP val1
```

Examples:

```text
10 3 -  => 10 - 3
10 3 /  => 10 / 3
```

---

# Variable Roles

| Variable | What it Stores | What it Represents |
|----------|----------|----------|
| stack | Evaluated values | Active subexpressions |
| s | Current token | Current input symbol |
| val1 | First pop | Right operand |
| val2 | Second pop | Left operand |

---

# Visual Trace

Input:

```text
["2","1","+","3","*"]
```

| Token | Stack |
|---------|---------|
| 2 | [2] |
| 1 | [2,1] |
| + | [3] |
| 3 | [3,3] |
| * | [9] |

Result:

```text
9
```

---

# Edge Cases

## Negative Division

```text
-10 / 6
```

Expected:

```text
-1
```

Use:

```python
int(a / b)
```

Not:

```python
a // b
```

because:

```python
-10 // 6
```

returns:

```text
-2
```

---

# Bugs I Actually Made

## Bug 1

Used:

```python
stack.push()
```

instead of:

```python
stack.append()
```

Type: Implementation Bug

---

## Bug 2

Used:

```python
s != "+" or s != "-" or s != "*" or s != "/"
```

Condition was always true.

Type: Logic Bug

---

## Bug 3

Wrong subtraction operand order.

Used:

```python
val1 -= val2
```

Needed:

```python
val2 -= val1
```

Type: Logic Bug

---

## Bug 4

Wrong division operand order.

Type: Logic Bug

---

## Bug 5

Pushed wrong variable back onto stack after computation.

Type: Logic Bug

---

## Bug 6

Did not truncate division result correctly.

Type: Logic Bug

---

# Interview Explanation (30 Seconds)

This is a stack problem because Reverse Polish Notation is a postfix expression format. We process tokens left-to-right. Numbers are pushed onto the stack. When an operator appears, we pop the right operand first and the left operand second, compute the result, and push it back. Each token is processed once, giving O(n) time and O(n) space complexity.

---

# Interview Explanation (2 Minutes)

Reverse Polish Notation places operators after their operands, making a stack the natural data structure for evaluation.

Invariant:

```text
After processing any token,
the stack contains the values of all fully evaluated subexpressions.
```

Numbers are pushed directly onto the stack.

When an operator is encountered, two values are popped. The first pop is the right operand and the second pop is the left operand. This ordering is critical for subtraction and division.

The result is computed and pushed back onto the stack, allowing larger expressions to be built incrementally.

Division requires truncation toward zero, which is achieved with:

```python
int(left / right)
```

instead of floor division.

Every token is processed exactly once.

Complexity:
- Time: O(n)
- Space: O(n)

Tradeoffs:
- Stack solution is simple and optimal.
- Requires careful operand ordering.
- Explicit version is easier to debug than compact one-liners.

---

# Complexity Summary

| Metric | Complexity |
|----------|----------|
| Time | O(n) |
| Space | O(n) |

---

# Mental Model

Whenever you see "Reverse Polish Notation", "Postfix Expression", "Evaluate Expression", or "Calculator", think Stack. Numbers wait on the stack until an operator arrives. The operator consumes the two most recent values, creates a new value, and pushes it back. The key rule is: first pop = right operand, second pop = left operand.
