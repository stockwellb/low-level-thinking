---
tags:
  - type/article
  - topic/boundaries
  - concept/zero-based-indexing
  - concept/pointer-arithmetic
  - first-principles
---
# Indexing

## Arrays

An array is a single block of memory. Its elements are the same size and sit one after another, starting at a fixed address called the base.

Reading the first two elements:

```c
int array[] = {1, 2, 3, 4}; // array names the base of the block

int first  = array[0]; // base + 0 * sizeof(int)
int second = array[1]; // base + 1 * sizeof(int)
```

## Raw Memory

`malloc` returns a pointer to raw memory, so the same block without the array type reads the same way:

```c
int *ptr = malloc(sizeof(int) * 4); // room for 4 ints

int first  = *ptr;       // base + 0 * sizeof(int)
int second = *(ptr + 1); // base + 1 * sizeof(int)
```

`array[i]` is defined as `*(array + i)`: take the base, move forward `i` elements, and read what is there. `array[0]` is `*(array + 0)`, the base unchanged.

## Distance vs Counting

When we use an index, we are really measuring a distance: how far an element is from the base. A count, usually written `len`, measures the total: how many elements there are. Index and count describe the same array in two ways, and most indexing mistakes come from using one where the other is meant.

An array of `len` elements has indices `0` through `len - 1`:
* The first element has index `0`.
* The last element has index `len - 1`.
* Index `len` is one step past the last element. Nothing is stored there.

So `len` is also the first index that is out of bounds.

## Bounds Checking

Because `len` is the first invalid index, the check rejects `len` and above:

```c
if (index >= len) {
    // out of bounds
}
```

Using `>` would let `index == len` through, which reads one element past the end. That is the same boundary that separates access from insertion in [[Ranges]].

## What Follows

Once you read an index as a distance from the base, the usual rules are consequences, not conventions:

| Rule | Reason |
|---|---|
| First element is index `0` | it sits at the base, zero elements away |
| Last element is `len - 1` | `len` elements have indices `0` through `len - 1` |
| Valid indices are `[0, len)` | indices `0` up to but not including `len` |
| Loop condition is `i < len` | stop when the index reaches `len` |
| Bounds check is `index >= len` | `len` is the first index past the last element |

Each row restates one fact: `len` elements have indices `0` through `len - 1`, so `len` is one past the end.

