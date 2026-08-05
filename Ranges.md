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
    // visit each insertion point, including the tail
}
```

In this example, `start = 0` and `end = 4`. The `4` is inclusive and represents a valid *insertion point* — not a valid *access* index. If this maps to a buffer with 4 existing items, a closed range loop gives you 5 total iterations (`0, 1, 2, 3, 4`): the 4 positions between and before the existing items, plus the appending slot at the tail. (For reading the 4 existing items you would still use the half-open `i < 4`, since index `4` is out of bounds for access.)

### Advantages
While half-open ranges dominate raw indexing, closed ranges provide essential physical and mathematical guardrails in low-level systems: 

* **Reaches the top of a fixed-width type:** A half-open loop over every value of a type needs `end = max + 1`, which the type cannot represent. To visit all 256 values of a `uint8_t`, `i < 256` never terminates because `256` overflows to `0`, so the counter never reaches it. A closed range expresses the intent directly as `i <= 255`, though note the counter itself must be *wider* than the range (e.g. `int i`) — otherwise `i++` wraps from `255` back to `0` and even the closed loop spins forever.
* **Protects memory layout during insertion:** By allowing `index <= length`, it defines a precise boundary where you can safely overwrite an existing slot or append right at the tail without leaving illegal, uninitialized gaps in your memory blocks.
* **Maps perfectly to human data:** It natively aligns with real-world, 1-indexed human systems like calendar dates, game dice, or UI pagination screens where the ending value is fully intended to be interactive.
* **Stays symmetric under negation:** Closed intervals map onto themselves when flipped or negated (e.g. `[-5, 5]` mirrors to `[-5, 5]`), whereas the half-open `[-5, 5)` becomes `(-5, 5]` — preventing the shifted boundaries that cause off-by-one slips during spatial transformations.

## Choosing Between Them

The two ranges are really the same idea seen through one character: the comparison operator. Placed side by side over a buffer of 4 items, the difference is stark:

```c
// Half-open: `<` guards ACCESS — every index touches a real element.
for (int i = 0; i < len; i++) {
    array[i] = 0;              // i = 0, 1, 2, 3   (4 iterations)
}

// Closed: `<=` reaches the TAIL — the extra step is the append slot.
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

