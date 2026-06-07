# 238. Product of Array Except Self

## Problem

Given an integer array `nums`, return an array `answer` such that:

```text
answer[i] = product of all the elements of nums except nums[i]
```

You must solve it without using division and in `O(n)` time.

---

# My Learning Journey

I initially approached the problem by computing the total product of the array and dividing by the current element.

```python
prod = 1
for n in nums:
    prod *= n

answer[i] = prod // nums[i]
```

While this works for arrays without zeros, it breaks when zeros are present and does not follow the intended interview approach.

After debugging, I explored the idea of building products from the left and right sides separately.

My first few attempts incorrectly used:

```python
nums[i] * nums[i-1]
```

which only computes products of neighboring elements rather than cumulative products.

Through debugging and tracing examples such as:

```python
[1,2,3,4]
```

I realized the important distinction between:

```text
Neighbor Product
```

and

```text
Prefix / Postfix Product
```

I then defined:

```text
prefix[i]  = product of everything left of i
postfix[i] = product of everything right of i
```

Once those definitions were clear, the implementation became much easier.

I also encountered an off-by-one bug when iterating backwards:

```python
range(len(nums)-2, 0, -1)
```

which skipped index `0`.

The fix was:

```python
range(len(nums)-2, -1, -1)
```

After correcting the prefix and postfix arrays, I successfully produced an accepted solution.

---

## Attempt 1

### Idea

Compute the total product and divide by the current element.

### Bug Found

Fails when zeros exist in the input.

Example:

```python
[1,2,0,4]
```

Also does not follow the intended interview solution.

### What I Learned

Division is not robust for this problem.

---

## Attempt 2

### Idea

Use left and right products.

### Bug Found

Used:

```python
nums[i] * nums[i-1]
```

which only computes adjacent products rather than cumulative products.

### What I Learned

Prefix products must accumulate all previous values.

---

## Accepted Solution (My Solution)

```python
class Solution:
    def productExceptSelf(self, nums: List[int]) -> List[int]:
        prod = []
        prod.append(1)

        for i in range(1, len(nums)):
            prod.append(nums[i - 1] * prod[i - 1])

        rProd = [1] * len(nums)
        rProd[len(nums) - 1] = 1

        for i in range(len(nums) - 2, -1, -1):
            rProd[i] = rProd[i + 1] * nums[i + 1]

        for i in range(len(nums)):
            prod[i] *= rProd[i]

        return prod
```

---

## NeetCode Solution

```python
class Solution:
    def productExceptSelf(self, nums: List[int]) -> List[int]:
        res = [1] * len(nums)

        prefix = 1
        for i in range(len(nums)):
            res[i] = prefix
            prefix *= nums[i]

        postfix = 1
        for i in range(len(nums) - 1, -1, -1):
            res[i] *= postfix
            postfix *= nums[i]

        return res
```

### Comparison

My Solution:

- Uses a prefix array and a postfix array.
- Easy to understand and debug.
- Uses extra space for the postfix array.

NeetCode Solution:

- Stores prefix products directly in the result array.
- Uses a single running postfix variable.
- Achieves O(1) extra space (excluding output).

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

---

## NeetCode Solution

### Time Complexity

```text
O(n)
```

### Space Complexity

```text
O(1) extra space
```

---

# Key Takeaways

- Prefix products and postfix products are a powerful interview pattern.
- Neighbor products are not the same as cumulative products.
- Clearly defining what each array represents makes debugging much easier.
- Off-by-one errors frequently occur when iterating backwards.
- The NeetCode optimization removes the need for a separate postfix array.
- Understanding the reasoning behind the pattern is more valuable than memorizing the final code.
