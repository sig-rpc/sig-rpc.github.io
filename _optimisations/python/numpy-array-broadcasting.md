---
level: 1
published: true
authors: Robert Chisholm

name: Array Broadcasting (Vectorisation)
language: [Python]
subcategory: [NumPy]
tags: [array, maths, loops]
---

The manner by which NumPy stores data in arrays enables it's functions to utilise array broadcasting (more broadly known as vectorisation), whereby the processor executes one instruction across multiple variables simultaneously, for every mathematical operation between arrays. Array broadcasting can perform mathematical operations many times faster, however it requires using supported functions.

<!--more-->

### Auto Parallel NumPy

<!-- https://superfastpython.com/multithreaded-numpy-functions/ -->
Additionally, NumPy functions which support array broadcasting can sometimes take advantage of auto parallelisation, particularly on HPC systems. These functions are typically backed by BLAS and LAPACK, it's [not well documented](https://superfastpython.com/multithreaded-numpy-functions/) which functions support auto parallelisation, but they most correspond to [linear algerbra operations](https://numpy.org/doc/stable/reference/routines.linalg.html).

The auto-parallelisation of these functions is hardware dependant, so you won't always automatically get the additional benefit of parallelisation. However, HPC systems should be primed to take advantage, so try increasing the number of cores you request when submitting your jobs and see if the performance improves.

### `vectorise()`

NumPy provides a `vectorize()` function for operating over it's arrays.

This doesn't actually make use of processor-level vectorisation, so won't afford a speed up. From NumPy's [documentation](https://numpy.org/doc/stable/reference/generated/numpy.vectorize.html):

> The `vectorize` function is provided primarily for convenience, not for performance. The implementation is essentially a for loop.

## Example Code

The below example computes the dot product of an array length 1 million.

```python
from timeit import timeit

N = 1000000  # Number of elements in list

gen_list = f"ls = list(range({N}))"
gen_array = f"import numpy;ar = numpy.arange({N}, dtype=numpy.int64)"

# List comprehension to compute
py_sum_ls = "sum([i*i for i in ls])"
# Vectorised multiplication
py_sum_ar = "sum(ar*ar)"
# Vectorised multiplication and sum
np_sum_ar = "numpy.sum(ar*ar)"
# Specialised vectorised dot product
np_dot_ar = "numpy.dot(ar, ar)"
# Numpy vectorise equivalent of py_sum_ar/py_sum_ls
np_vec_ar = "sum(numpy.vectorize(<todo lambda>)(ar))"

repeats = 1000
print(f"python_sum_list: {timeit(py_sum_ls, setup=gen_list, number=repeats):.2f}ms")
print(f"python_sum_array: {timeit(py_sum_ar, setup=gen_array, number=repeats):.2f}ms")
print(f"numpy_sum_array: {timeit(np_sum_ar, setup=gen_array, number=repeats):.2f}ms")
print(f"numpy_dot_array: {timeit(np_dot_ar, setup=gen_array, number=repeats):.2f}ms")
print(f"numpy_vec_array: {timeit(np_vec_ar, setup=gen_array, number=repeats):.2f}ms")
```

* `python_sum_list` uses list comprehension to perform the multiplication, followed by the Python core `sum()`. This comes out at 46.93ms
* `python_sum_array` instead directly multiplies the two arrays, taking advantage of NumPy's vectorisation. But uses the core Python `sum()`, this comes in slightly faster at 33.26ms.
* `numpy_sum_array` again takes advantage of NumPy's vectorisation for the multiplication, and additionally uses NumPy's `sum()` implementation. These two rounds of vectorisation provide a much faster 1.44ms completion.
* `numpy_dot_array` instead uses NumPy's `dot()` to calculate the dot product in a single operation. This comes out the fastest at 0.29ms, 162x faster than `python_sum_list`. 

```output
python_sum_list: 46.93ms
python_sum_array: 33.26ms
numpy_sum_array: 1.44ms
numpy_dot_array: 0.29ms
```

## The Technical Detail

The vector instructions, which NumPy's array broadcasting take advantage of, enable a CPU to apply the same operation to multiple data elements in parallel with a single thread.

Modern CPUs use SIMD (Single Instruction, Multiple Data) instructions to operate on several values packed into a register. Since a typical CPU cache line is 64 bytes, and standard data types like 32-bit or 64-bit floats and integers are 4 or 8 bytes each, 8–16 values can fit within a single cache line and be processed together by a single vector instruction.

However, to take advantage of this, the data must be aligned in memory. Laid out such that it fits neatly into the expected cache line boundaries. Default memory allocations in Python don’t guarantee this kind of alignment, as doing so for all objects would waste memory.

NumPy arrays, in contrast, are explicitly designed for numerical performance. They allocate memory with alignment guarantees, ensuring that vector instructions can be used.
