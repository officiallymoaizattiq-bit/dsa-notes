# 271. Encode and Decode Strings

## Problem

Design an algorithm to encode a list of strings into a single string so that it can be transmitted over a network and later decoded back into the exact original list.

The solution must work for:

- Empty strings
- Strings containing special characters
- Strings containing the delimiter itself
- Variable-length strings

---

# My Learning Journey

This problem initially confused me because the input and output appeared to be the same list of strings.

The key realization was that the challenge is not about returning the same list. The challenge is about converting a **list of strings** into **one single string** and then reconstructing the original list perfectly.

## First Attempt

I started by encoding each string as:

```text
#<length><string>
```

Example:

```text
#5Hello#5World
```

My initial decoder simply looked for `#` and attempted to slice characters after it.

### What I learned

- The decoder must actually use the stored length.
- Finding a delimiter is not enough.
- Serialization requires enough metadata to reconstruct the original data.

---

## Second Attempt

I updated the decoder to read:

```python
length = int(s[i + 1])
```

and then extract the next `length` characters.

### Bug Found

This only worked for single-digit lengths.

Example:

```text
#12abcdefghijkl
```

The decoder would read:

```python
1
```

instead of:

```python
12
```

### What I learned

Lengths themselves can have variable length.

---

## Third Attempt

I switched from:

```python
for i in range(len(s))
```

to:

```python
while i < len(s)
```

and began moving a pointer through the encoded string.

### What I learned

Pointer-based parsing is a powerful pattern used in:

- String parsing
- Two pointers
- Sliding window
- Linked lists

---

## My Final Encoding Design

Instead of using the common NeetCode format, I created my own encoding scheme.

For each string:

```text
#<number_of_digits_in_length><length><string>
```

Example:

```text
Hello
```

Length:

```text
5
```

Encoded as:

```text
#15Hello
```

because:

- Length = 5
- Number of digits in "5" = 1

Another example:

```text
abcdefghijkl
```

Length:

```text
12
```

Encoded as:

```text
#212abcdefghijkl
```

because:

- Length = 12
- Number of digits in "12" = 2

---

## Debugging Process

Several indexing mistakes had to be fixed.

### Bug 1

I assumed the string always started at:

```python
i + 3
```

which only works when the length has one digit.

### Fix

The start index must account for:

- `#`
- the digit count
- the full length field

---

### Bug 2

My pointer jump calculation ignored the size of the length field.

I corrected the pointer movement so it skips:

- metadata
- length field
- actual string contents

and lands directly on the next encoded record.

---

## Accepted Solution

```python
class Solution:

    def encode(self, strs: List[str]) -> str:
        String=""
        for s in strs:
            lenStr=str(len(s))
            String=String + "#" + str(len(lenStr)) + lenStr + s
        return String

    def decode(self, s: str) -> List[str]:
        StringList=[]
        i=0

        while i < len(s):
            if s[i] == "#":
                numOfDigits=int(s[i+1])
                length=int(s[i+2:i+2+numOfDigits])

                StringList.append(
                    s[
                        i+1+numOfDigits+1 :
                        i+1+numOfDigits+1+length
                    ]
                )

            i=i+1+numOfDigits+1+length

        return StringList
```

---

# NeetCode Solution

The standard NeetCode approach stores:

```text
<length>#<string>
```

Example:

```text
5#Hello5#World
```

During decoding:

1. Read digits until `#`.
2. Convert those digits into a length.
3. Extract the next `length` characters.
4. Move the pointer forward.
5. Repeat.

```python
def encode(self, strs):
    res = ""

    for s in strs:
        res += str(len(s)) + "#" + s

    return res


def decode(self, string):
    res = []
    i = 0

    while i < len(string):
        j = i

        while string[j] != "#":
            j += 1

        length = int(string[i:j])

        res.append(
            string[j + 1 : j + 1 + length]
        )

        i = j + 1 + length

    return res
```

---

# Complexity Analysis

## My Solution

### Encode

Time Complexity:

```text
O(n)
```

where `n` is the total number of characters across all strings.

### Decode

Time Complexity:

```text
O(n)
```

### Space Complexity

```text
O(n)
```

for the encoded string and decoded output.

---

# Key Takeaways

- Serialization means converting structured data into a transferable format.
- Delimiters alone are often insufficient because the delimiter may appear in the data.
- Length-prefix encoding is a common and reliable technique.
- Pointer-based parsing is often cleaner than scanning character-by-character.
- Multiple valid serialization formats can exist as long as encoding and decoding agree on the same protocol.
