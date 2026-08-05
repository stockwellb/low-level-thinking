# Roadmap

A list of short articles, each explaining the reasoning underneath a rule so the rule can be derived rather than memorized. Some of these ideas are rarely given a name; others, such as how `const` binds in a declaration, are well defined but usually taught as rules to memorize. In both cases, the article aims at the underlying reason.

The standard to meet is the one *Ranges* set: define the concept plainly, show the least code needed, and leave the reader able to reconstruct the rule from the idea. C is the lens, not the subject.

This roadmap is the source of truth for the article plan. Each entry names a concept, states its core idea, and lists the terms the article should teach.

**Code.** Examples are small and self-contained. Each shows one idea in a few lines of plain C, with no surrounding function or struct unless the idea needs it. Write the snippet for the point being made rather than lifting it from a larger program, which carries in names and error handling that bury the idea.

## Conventions

1. Lead with the plain-language idea, then the term or notation.
2. One small, self-contained code block per idea. A few lines of plain C; add a function or struct only if the idea requires it.
3. Use a table or a paired snippet when two things are one idea seen from two sides.
4. Close with a way to apply the idea, usually a question to ask rather than an answer to recall.
5. Name the concept explicitly so the reader has a term to search and reuse.
6. No em dashes.

---

## Part I: Counting and Boundaries

**1. Indexing** (written, `Indexing.md`)
An index is an offset, the distance from the start. Because `array[i]` is defined as `*(array + i)`, zero-based indexing is a consequence rather than a choice, and the rules for last element, valid range, and bounds checks all follow from it. Names: zero-based indexing, offset addressing. Source: Dijkstra, EWD831.

**2. Ranges** (written, `Ranges.md`)
Half-open and closed ranges, and how the difference between them reduces to `<` versus `<=`.

**3. Past-the-end Pointers**
The address one past the last element is valid to form and compare but not to dereference. That guarantee is what makes loop terminators, ring-buffer wraps, and end iterators well defined. Names: one-past-the-end pointer, sentinel. Pairs with Ranges, since the excluded end of a half-open range is this address.

## Part II: Reading the Types

**4. Pointer Arithmetic**
`p + 1` advances by `sizeof(*p)` bytes, not one byte, because the pointer's type carries the element size. This is the scaling that `Indexing` relies on. Two consequences that are often skipped: `void *` has no element size and so has no defined arithmetic, and the declaration ladder is uniform, with `void **` being a pointer to `void *` in the same way `int **` is a pointer to `int *`. Names: typed pointer arithmetic, pointer stride.

**5. const Declarations**
`const int *` and `int * const` are read, not memorized. `const` applies to the token on its left, or to the token on its right when it is leftmost, and a declaration read right to left states itself: "pointer to const int" versus "const pointer to int." Names: declaration reading, top-level versus low-level const.

## Part III: Numbers That Wrap

**6. Integer Wraparound**
Fixed-width integers wrap modulo their range. This has a hazard and a use. The hazard: `size_t` is unsigned, so `len - 1` when `len == 0` wraps to a very large number rather than going negative, so the empty case must be checked before the subtraction. The use: `(w - r + cap) % cap` uses the same wraparound to normalize a ring index back into `[0, cap)`. Names: unsigned wraparound, modular arithmetic, integer promotion.
Aside to include: a loop over a whole fixed-width type needs a counter wider than the type. `for (uint8_t i = 0; i <= 255; i++)` never ends, because the last increment wraps `255` back to `0`. This is the subtlety left out of the closed-range advantage in Ranges.

## Part IV: Pointers and Memory

**7. Indirect Pointers**
To rewrite a link, hold the address of the pointer that points in (`T**`) rather than the node it points at. The head case and the interior case then use the same code, with no separate `prev` tracking. Names: pointer to pointer, indirect pointer.

**8. Overlapping Copies**
`memmove` and `memcpy` differ over aliasing. `memcpy` may overwrite source bytes it has not read yet when the regions overlap, and shifting elements within one array is the overlapping case. Names: overlapping copy, aliasing.

## Part V: Ownership and Lifecycle

**9. Null-safe Teardown**
`free(NULL)` is defined to do nothing, so a destroy function needs one guard for the handle and can then free its members directly. Making an operation defined over every input, including null, removes scattered null checks. Names: guard clause, total function.

**10. Error-path Cleanup**
A constructor that fails partway owns a half-built object that no other code knows about, so it must release what it has allocated, in reverse order, before returning the error. This is the manual form of a destructor. Names: error-path unwinding, cascading cleanup.

**11. Amortized Growth**
Growing a buffer by a constant amount copies O(n squared) over its life; growing by a factor makes reallocations rare and brings the total copying to O(n), so each append is O(1) amortized. Names: amortized analysis, geometric growth.

## Part VI: Representation and State

**12. Opaque Types**
Placing the struct definition in the `.c` file and only its name in the `.h` file makes it an incomplete type, which the compiler will not allow callers to dereference. This is how C enforces private fields. Names: opaque type, incomplete type, information hiding.

**13. State Disambiguation**
When one representation stands for two different states, such as `wptr == rptr` meaning both empty and full, or `value == NULL` meaning both a stored null and an absent entry, the distinguishing information is not present in the value and must be stored separately. Names: disambiguation bit, discriminator, tagged union.

---

## Backlog

Concepts named but not yet placed.

- Invariants: the property a structure maintains between calls, and what tree rotations restore.
- Recursion versus an explicit stack: the same traversal with the call stack doing the bookkeeping or the code doing it.
- Tombstones: why a deleted slot cannot simply become empty in open addressing.
- Intrusive versus non-intrusive nodes: links stored inside the payload, as in an LRU cache whose hash table and list share nodes.
- Bit sets: many booleans packed into one word, as in a bloom filter.
- Idempotence: an operation that is safe to apply more than once.
- Copy-on-write: deferring a copy until a write requires it.
- Structural sharing: reusing nodes or subtrees across versions instead of copying them.
- Memoization: storing a result so a repeated computation runs once.
- Arity: the number of operands an operation takes.
- Strict aliasing: the type-based assumption the optimizer is allowed to make, behind article 8.

---

## Order to Write In

The order follows concept dependencies rather than difficulty. Write 1, 2, 3 first, since they build one boundary model. Then 4 and 5, which cover how types and declarations read and repay the pointer-arithmetic point left open in article 1. Then 6, which stands alone. Then the pointer pair 7 and 8, the lifecycle set 9, 10, 11, and the representation pair 12 and 13. Each reads on its own, but in this order the earlier terms are available to the later articles.
