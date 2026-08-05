# Half-open Ranges

Half-open ranges are intervals that include the beginning value but exclude the ending value. They are standard interval notation, typically written as: `[start, end)`.

Another common way to refer to this type of range is "exclusive," as the range excludes the final value.

### Loop Example
```c
for (int i = 0; i < 4; i++) { 
    // Loops exactly 4 times
}
```
In this example, `start = 0` and `end = 4`. The `4` is exclusive and sits just beyond the valid range. If this mapped to an array of size 4, the indices are perfectly bound:
* `array[0]`
* `array[1]`
* `array[2]`
* `array[3]`

## Advantages

The primary advantage of a half-open range is that it removes a whole class of bugs called **off-by-one errors**. This idiom provides several elegant mechanical advantages:

* **Counting is pure subtraction:** Calculating size is simply `end - start = length`. In our example: `4 - 0 = 4`.
* **Adjacency without gaps:** You can split `[0, 10)` perfectly in half as `[0, 5)` and `[5, 10)` without overlapping numbers.
* **Empty states are elegant:** An empty range is naturally expressed when boundaries match; `[3, 3)` safely contains zero elements.
* **Length becomes a sentinel:** The `len` variable acts as a perfect out-of-bounds stop-sign for pointers and indexing.
* **Asymmetric Guardrails:** In this model, `<` guards access within the valid range, while `<=` guards insertions at or beyond the boundary.

### Guard Examples
```c
for (int i = 0; i < 4; i++) { 
    // access to each item
}

for (int i = 0; i <= 4; i++) { 
    // access to each insert slot
}
```
