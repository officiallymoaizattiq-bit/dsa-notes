# 36. Valid Sudoku

## Problem

Determine if a 9×9 Sudoku board is valid.

Rules:

- Each row may contain a digit at most once.
- Each column may contain a digit at most once.
- Each 3×3 box may contain a digit at most once.
- `"."` represents an empty cell and should be ignored.

---

# My Learning Journey

This problem was very different from the earlier array/hashmap problems.

At first, I tried using a frequency table and checking whether every digit appeared exactly once.

### Important Realization

Sudoku does **not** require:

```text
1 appears once
2 appears once
...
9 appears once
```

A valid row can look like:

```text
5 3 . . 7 . . . .
```

Many digits never appear.

The real rule is:

```text
No digit may appear more than once.
```

That realization completely changed the approach.

---

## Attempt 1

### Idea

Use a frequency table for each row and count occurrences.

### Bugs Found

- Used incorrect indexing such as `board[i[j]]`.
- Used undefined variables.
- Checked whether digits appeared exactly once.
- Condition was logically always true.

### What I Learned

Instead of counting everything, ask:

```text
Have I seen this value before?
```

This naturally leads to using a set.

---

## Attempt 2

### Row Validation

Created a set for each row.

Process:

1. Ignore `"."`
2. If digit already exists in set → invalid
3. Otherwise add it

### What I Learned

A set is perfect for duplicate detection because membership checks are very fast.

---

## Attempt 3

### Column Validation

Reused the exact same logic as rows.

Only the traversal changed.

### Key Insight

The validation logic never changed:

```text
if value already seen:
    duplicate found
```

Only the traversal changed.

---

## Attempt 4

### 3×3 Box Validation

This was the hardest part.

The challenge was not sets.

The challenge was understanding how to traverse subgrids.

---

# Visual Sudoku Map

```text
┌─────────────┬─────────────┬─────────────┐
│ (0,0)(0,1)(0,2) │ (0,3)(0,4)(0,5) │ (0,6)(0,7)(0,8) │
│ (1,0)(1,1)(1,2) │ (1,3)(1,4)(1,5) │ (1,6)(1,7)(1,8) │
│ (2,0)(2,1)(2,2) │ (2,3)(2,4)(2,5) │ (2,6)(2,7)(2,8) │
├─────────────┼─────────────┼─────────────┤
│ (3,0)(3,1)(3,2) │ (3,3)(3,4)(3,5) │ (3,6)(3,7)(3,8) │
│ (4,0)(4,1)(4,2) │ (4,3)(4,4)(4,5) │ (4,6)(4,7)(4,8) │
│ (5,0)(5,1)(5,2) │ (5,3)(5,4)(5,5) │ (5,6)(5,7)(5,8) │
├─────────────┼─────────────┼─────────────┤
│ (6,0)(6,1)(6,2) │ (6,3)(6,4)(6,5) │ (6,6)(6,7)(6,8) │
│ (7,0)(7,1)(7,2) │ (7,3)(7,4)(7,5) │ (7,6)(7,7)(7,8) │
│ (8,0)(8,1)(8,2) │ (8,3)(8,4)(8,5) │ (8,6)(8,7)(8,8) │
└─────────────┴─────────────┴─────────────┘
```

---

# Key Breakthrough

The 9 boxes start at:

```text
(0,0) (0,3) (0,6)

(3,0) (3,3) (3,6)

(6,0) (6,3) (6,6)
```

This can be generated using:

```python
range(0, 9, 3)
```

which produces:

```text
0, 3, 6
```

---

# My Accepted Solution

```python
class Solution:
    def isValidSudoku(self, board: List[List[str]]) -> bool:
        for row in range(9):
            seen = set()
            for val in range(9):
                if board[row][val] != ".":
                    if board[row][val] in seen:
                        return False
                    else:
                        seen.add(board[row][val])

        for col in range(9):
            seen = set()
            for val in range(9):
                if board[val][col] != ".":
                    if board[val][col] in seen:
                        return False
                    else:
                        seen.add(board[val][col])

        for startRow in range(0, 9, 3):
            for startCol in range(0, 9, 3):
                seen = set()

                for row in range(startRow, startRow + 3):
                    for col in range(startCol, startCol + 3):
                        if board[row][col] != ".":
                            if board[row][col] in seen:
                                return False
                            else:
                                seen.add(board[row][col])

        return True
```

---

# NeetCode Solution

```python
class Solution:
    def isValidSudoku(self, board: List[List[str]]) -> bool:
        cols = collections.defaultdict(set)
        rows = collections.defaultdict(set)
        squares = collections.defaultdict(set)

        for r in range(9):
            for c in range(9):

                if board[r][c] == ".":
                    continue

                if (
                    board[r][c] in rows[r]
                    or board[r][c] in cols[c]
                    or board[r][c] in squares[(r // 3, c // 3)]
                ):
                    return False

                rows[r].add(board[r][c])
                cols[c].add(board[r][c])
                squares[(r // 3, c // 3)].add(board[r][c])

        return True
```

---

# Deep Understanding of the NeetCode Solution

The NeetCode solution does not scan rows, columns, and boxes separately.

Instead:

```text
It scans every cell exactly once.
```

Think of each digit being registered in three places:

1. Its row
2. Its column
3. Its box

---

## Visual Example

Suppose:

```text
board[4][7] = "5"
```

Then NeetCode stores:

```text
Row 4 contains 5
Column 7 contains 5
Box (?, ?) contains 5
```

---

## Understanding `(r // 3, c // 3)`

This is the hardest part.

Integer division groups cells into boxes.

Examples:

```text
(0,0) -> (0,0)
(1,2) -> (0,0)
(2,1) -> (0,0)
```

Top-left box.

---

```text
(0,5) -> (0,1)
(1,4) -> (0,1)
(2,3) -> (0,1)
```

Top-middle box.

---

```text
(7,8) -> (2,2)
```

Bottom-right box.

---

## Complete Box Map

```text
Rows 0-2 and Cols 0-2 => box (0,0)
Rows 0-2 and Cols 3-5 => box (0,1)
Rows 0-2 and Cols 6-8 => box (0,2)

Rows 3-5 and Cols 0-2 => box (1,0)
Rows 3-5 and Cols 3-5 => box (1,1)
Rows 3-5 and Cols 6-8 => box (1,2)

Rows 6-8 and Cols 0-2 => box (2,0)
Rows 6-8 and Cols 3-5 => box (2,1)
Rows 6-8 and Cols 6-8 => box (2,2)
```

---

# Comparison

## My Solution

Pros:

- Easier to visualize.
- Easier to learn.
- Separates rows, columns, and boxes.

Cons:

- Traverses the board three times.

---

## NeetCode Solution

Pros:

- Single traversal.
- Elegant.
- More interview-optimized.

Cons:

- Harder to understand initially.
- Requires understanding hash maps of sets.
- Requires understanding `(r // 3, c // 3)`.

---

# Complexity Analysis

## My Solution

Time Complexity:

O(81)

Space Complexity:

O(1)

---

## NeetCode Solution

Time Complexity:

O(81)

Space Complexity:

O(1)

---

# Key Takeaways

- Sudoku is a duplicate-detection problem.
- Sets are excellent for detecting duplicates.
- Traversal and validation are separate concepts.
- The hardest part was learning 3×3 box traversal.
- `range(0, 9, 3)` generates box starting positions.
- `r // 3` and `c // 3` identify which box a cell belongs to.
- My solution is easier to understand.
- The NeetCode solution performs all checks in one pass.
