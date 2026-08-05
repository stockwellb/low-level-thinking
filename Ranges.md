---
tags:
  - type/article
  - topic/boundaries
  - concept/half-open-range
  - concept/off-by-one
  - first-principles
---
# Ranges

## Half-open Ranges

Half-open ranges are intervals that include the beginning value but exclude the ending value. They are standard interval notation, typically written as: `[start, end)`.

Another common way to refer to this type of range is "exclusive," as the range excludes the final value.

### Loop Example
```c
for (int i = 0; i < 4; i++) { 
    // access to each item
}
```

In this example, `start = 0` and `end = 4`. The `4` is exclusive and sits just beyond the valid range. If this mapped to an array of size 4, the indices are perfectly bound:
* `array[0]`
* `array[1]`
* `array[2]`
* `array[3]`

### Advantages

The primary advantage of a half-open range is that it removes a whole class of bugs called **off-by-one errors**. This idiom provides several elegant mechanical advantages:

* **Counting is pure subtraction:** Calculating size is simply `end - start = length`. In our example: `4 - 0 = 4`.
* **Adjacency without gaps:** You can split `[0, 10)` perfectly in half as `[0, 5)` and `[5, 10)` without overlapping numbers.
* **Empty states are elegant:** An empty range is naturally expressed when boundaries match; `[3, 3)` safely contains zero elements.
* **Length becomes a sentinel:** The `len` variable acts as a perfect out-of-bounds stop-sign for pointers and indexing.

## Closed Ranges

Closed ranges are intervals that include the beginning value and include the ending value. They are standard interval notation, typically written as: `[start, end]`.

Another common way to refer to this type of range is "inclusive," as the range includes the final value.

### Loop Example
```c
for (int i = 0; i <= 4; i++) { 
    // each value from start to end
}
```

In this example, `start = 0` and `end = 4`. The `4` is inclusive and part of the valid range. Unlike the half-open version, the loop runs five times:
* `i = 0`
* `i = 1`
* `i = 2`
* `i = 3`
* `i = 4`

### Advantages

Closed ranges are the natural choice when the final value is meaningful. They offer a few advantages of their own:

* **Represents the maximum value:** A closed range can name a type's largest value as its endpoint, like `[0, 255]` for a `uint8_t`; the half-open form would need `256`, which the type cannot hold.
* **Includes the endpoint for insertion:** Allowing `index <= length` permits appending at the tail, so the position just past the last element is a valid target.
* **Maps to human counts:** It fits 1-indexed ranges like calendar dates or dice, where the final value is meant to be used.
* **Symmetric under negation:** A closed interval mirrors onto itself, so `[-5, 5]` stays `[-5, 5]` when flipped, unlike the half-open `[-5, 5)`.

## Choosing Between Them

The two ranges are really the same idea seen through one character: the comparison operator. Placed side by side over a buffer of 4 items, the difference is stark:

```c
// Half-open: `<` guards access. Every index touches a real element.
for (int i = 0; i < len; i++) {
    array[i] = 0;              // i = 0, 1, 2, 3   (4 iterations)
}

// Closed: `<=` reaches the tail. The extra step is the append slot.
for (int i = 0; i <= len; i++) {
    // i = 0, 1, 2, 3, 4       (5 iterations)
    // i < len  -> an existing element
    // i == len -> the one-past-the-end insertion point
}
```

| | Half-open `[start, end)` | Closed `[start, end]` |
|---|---|---|
| Guard | `<` | `<=` |
| Question it answers | "Is this a valid element?" | "Is this a valid position?" |
| Iterations over `len` | `len` | `len + 1` |
| Best for | Reading and iterating elements | Inserting, appending, spanning a full type |

The rule of thumb: reach for `<` when you are **touching elements**, and `<=` when you care about the **boundaries between and beyond them**.

