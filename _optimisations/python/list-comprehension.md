---
level: 1
published: true
authors: Robert Chisholm

name: List Comprehension
language: [Python]
subcategory: [Core]
tags: [list, loop]
---

[List comprehension](https://docs.python.org/3/tutorial/datastructures.html#list-comprehensions) (e.g. `[expression for item in iterable if condition == True]`) is faster than constructing a list with a loop. it can't be used for all list construction, such as where items depend on one another, but it should be used whenever possible.

<!--more-->

## Example Code

Rather than constructing a `list` with `append()` inside a loop

```py
squares = []
for x in range(100):
    squares.append(x**2)
```

Use a list comprehension

```py
squares = [x**2 for x in range(100)]
```

Whilst being more concise and readable, it is also typically faster. The speed-up will depend on the length of the list, and complexity of it's construction but is likely to be twice as fast.

List comprehension syntax can be nested to create two-dimensional lists, however using [`map()`](https://docs.python.org/3/library/functions.html#map) and [`zip()`](https://docs.python.org/3/library/functions.html#zip) may create more readable code in these scenarios.

<!-- Todo, can this be made into a WebASM example? -->
<details markdown="block">
<summary>Example Benchmark</summary>
The below code provides a simple benchmark of creating a list of 100,000 consecutive integers using a three approaches.

```py
from timeit import timeit

# Construct the list appending each item individually
def list_append():
    li = []
    for i in range(100000):
        li.append(i)

# Construct the list by preallocating it, before setting each item to the correct value
def list_preallocate():
    li = [0]*100000
    for i in range(100000):
        li[i] = i

# Construct the list using list comprehension
def list_comprehension():
    li = [i for i in range(100000)]

repeats = 1000
print(f"Append: {timeit(list_append, number=repeats):.2f}ms")
print(f"Preallocate: {timeit(list_preallocate, number=repeats):.2f}ms")
print(f"Comprehension: {timeit(list_comprehension, number=repeats):.2f}ms")
```

In testing this produced the following results

```
list_append: 3.50ms
list_preallocate: 2.48ms
list_comprehension: 1.69ms
```

Using list comprehension was over 2x faster than using a loop and `append()`.

</details>

## The Technical Detail

List comprehension is likely faster than using loops and append, because the syntax is more constrained allowing greater optimisation within the Python back-end.

### How Lists Are Implemented

Lists are implemented as a form of dynamic array found within many programming languages by different names (C++: `std::vector`, Java: `ArrayList`, R: `vector`, Julia: `Vector`).

They allow direct and sequential element access, with the convenience to append items.

This is achieved by internally storing items in a static array. This array however can be longer than the list, so the current length of the list is stored alongside the array. When an item is appended, the list checks whether it has enough spare space to add the item to the end. If it doesn’t, it will re-allocate a larger array, copy across the elements, and deallocate the old array. The item to be appended is then copied to the end and the counter which tracks the list’s length is incremented.

The amount the internal array grows by is dependent on the particular list implementation’s growth factor. CPython for example uses [`newsize + (newsize >> 3) + 6`](https://github.com/python/cpython/blob/a571a2fd3fdaeafdfd71f3d80ed5a3b22b63d0f7/Objects/listobject.c#L74), which works out to an over allocation of roughly ~12.5%.

![A graph demonstrating the number of resizes required based on the number of appends to a list within CPython. The plot is almost vertical for x < 100,000, before tapering off.](/assets/python/cpython_list_allocations.png)<br/>The relationship between the number of appends to an empty list, and the number of internal resizes in CPython.

This has two implications:

* If you are creating large static lists, they will use upto 12.5% excess memory.
* If you are growing a list with `append()`, there will be large amounts of redundant allocations and copies as the list grows.

Both of which will result in slower code. 

