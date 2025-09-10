---
level: 2
published: true
authors: Robert Chisholm

name: numpy.array.resize()
language: [Python]
subcategory: [NumPy]
tags: [list, array, append, resize]
---

NumPy's arrays are static arrays which, unlike core Python's lists, do not dynamically resize.
If you wish to append to a NumPy array, you must call `resize()` first, which can lead to performance issues.
If a user treats `array.resize` like `list.append`, resizing for each individual append, they will be performing significantly more copies and memory allocations and hence make your code slower.

<!--more-->

While resizing can be valid in some scenarios, it should generally be minimised.
Ideally `array.resize` should be used to increase (or decrease) capacity by many elements at once (similar to how a list work internally).

## Example Code

The below example sees lists and arrays constructed from `range(100000)`.

```python
from timeit import timeit
import numpy

N = 100000  # Number of elements in list/array

def list_comprehension():
    ls = [i for i in range(N)]

def list_append():
    ls = []
    for i in range(N):
        ls.append(i)

def array_resize():
    ar = numpy.zeros(1)
    for i in range(1, N):
        ar.resize(i+1)
        ar[i] = i

repeats = 1000
print(f"list_comprehension: {timeit(list_comprehension, number=repeats):.2f}ms")
print(f"list_append: {timeit(list_append, number=repeats):.2f}ms")
print(f"array_resize: {timeit(array_resize, number=repeats):.2f}ms")
```

Resizing a NumPy array was 8x slower than a list, and 13x slower than list comprehension.

```output
list_comprehension: 6.29ms
list_append: 9.68ms
array_resize: 82.66ms
```

## The Technical Detail

Resizing an array, allocates a new buffer in memory of the required size, copies the data across, and de-allocates the old storage.

In comparison a list, whilst backed by an array, performs greedy resizes. The length of the list visible to the user, does not reflect the length of the internal array. This allows it to simply store new items and increase it's length counter when appending. Only occasionally, does it need to perform an additional greedy resize. This greatly improves the append performance, at the cost of a slight memory overhead.
