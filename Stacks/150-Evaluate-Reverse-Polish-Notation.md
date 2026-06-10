# Problem

Evaluate an arithmetic expression given in Reverse Polish Notation (postfix notation).

Supported operators:
- +
- -
- *
- /

Division truncates toward zero.

# Pattern Recognition

## Recognition Signals
- Reverse Polish Notation (RPN)
- Postfix expression
- Evaluate expression
- Most recent operands are used first

## Pattern Used
- Stack

## Why This Pattern Fits
- Operators consume the most recently seen operands.
- Stack provides LIFO access.

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
Push numbers. For operators, pop right operand, pop left operand, compute, push result.

### Time Complexity
O(n)

### Space Complexity
O(n)

### Tradeoffs
- Explicit and easy to debug
- Slightly more verbose

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
