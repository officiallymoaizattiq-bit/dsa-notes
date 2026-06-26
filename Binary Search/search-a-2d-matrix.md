# Search a 2D Matrix (LeetCode 74)

## Problem

Given an `m x n` matrix where:
- each row is sorted in ascending order (left to right), and
- the first integer of each row is greater than the last integer of the previous row,

return `True` if `target` is in the matrix, else `False`. Target time complexity: `O(log(m * n))`.

## Key insight

Because the last element of every row is smaller than the first element of the next row, the matrix is just **one big sorted array that's been folded into rows**. That means binary search applies twice:

1. **Binary search the rows** to find the single row that *could* contain the target.
2. **Binary search inside that row** to find the target itself.

## Solution

```python
class Solution:
    def searchMatrix(self, matrix: List[List[int]], target: int) -> bool:
        ROWS, COLS = len(matrix), len(matrix[0])

        # Pass 1: binary search to find the candidate row
        top, bot = 0, ROWS - 1
        while top <= bot:
            row = (top + bot) // 2
            if target > matrix[row][-1]:      # target is below this row's range
                top = row + 1
            elif target < matrix[row][0]:     # target is above this row's range
                bot = row - 1
            else:                             # matrix[row][0] <= target <= matrix[row][-1]
                break

        if not (top <= bot):                  # loop exited without finding a candidate row
            return False

        # Pass 2: binary search within the candidate row
        row = (top + bot) // 2
        l, r = 0, COLS - 1
        while l <= r:
            m = (l + r) // 2
            if target > matrix[row][m]:
                l = m + 1
            elif target < matrix[row][m]:
                r = m - 1
            else:
                return True
        return False
```

## How it works

### Pass 1 — find the row

`top`/`bot` are the row bounds. At each step it checks the midpoint `row` against that row's **range** `[matrix[row][0], matrix[row][-1]]`:

- `target > matrix[row][-1]` → target is bigger than everything in this row, search lower rows (`top = row + 1`).
- `target < matrix[row][0]` → target is smaller than everything in this row, search higher rows (`bot = row - 1`).
- otherwise the target falls inside this row's range, so this is the only row it can be in → `break`.

If the loop ends without ever hitting the `else`, no row's range contained the target. The guard `if not (top <= bot): return False` catches that — when you break out of a `while top <= bot` loop normally (without `break`), `top > bot` is true, so this correctly bails when no candidate row exists.

### Pass 2 — find the value

`row = (top + bot) // 2` recomputes the candidate row, then it's a textbook binary search across that row's columns. Found → `return True`, otherwise the loop exhausts and `return False`.

## Complexity

- **Time:** `O(log m + log n)` = `O(log(m * n))`. One binary search over `m` rows, one over `n` columns.
- **Space:** `O(1)`. Only index variables, no extra structures.

## Note: the cleaner one-pass version

Since the matrix is logically a flat sorted array of length `m * n`, you can do it with a **single** binary search by mapping a flat index `i` to `(i // COLS, i % COLS)`:

```python
class Solution:
    def searchMatrix(self, matrix: List[List[int]], target: int) -> bool:
        ROWS, COLS = len(matrix), len(matrix[0])
        l, r = 0, ROWS * COLS - 1
        while l <= r:
            m = (l + r) // 2
            val = matrix[m // COLS][m % COLS]
            if target > val:
                l = m + 1
            elif target < val:
                r = m - 1
            else:
                return True
        return False
```

Same `O(log(m * n))` time, less code, no edge-case guard between passes. The two-pass version is worth understanding though — it's the natural setup for **LeetCode 240** (Search a 2D Matrix II), where rows and columns are sorted but rows *don't* chain together, so the flatten trick stops working.
