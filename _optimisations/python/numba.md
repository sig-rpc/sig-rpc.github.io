---
level: 3
published: true
authors: Nick Macey, Robert Chisholm

name: Use Numba to precompile and optimise Python functions 
language: [Python]
subcategory: [Numba]
tags: [compiler, parallel]
---

[Numba](numba.pydata.org) is an open-source Just In Time (JIT) compiler which converts Python functions into optimised machine code at runtime. At its simplest, it can be invoked by simply adding the `@njit` decorator from the `numba` package before your function declarations.

<!--more-->

Unless cached, the first time a function decorated with `@njit` is executed it will also incur the small compilation cost. Often this is still faster than the original Python code. Subsequent calls to the same function, will run the compiled function from Numba's cache and execute even faster.

Numba can't compile all Python code, it will sometimes lead to an error, but with it being so quick to try with a potential for high speedups. If profiling identifies an expensive function or block of code, maybe try wrapping it with Numba.

### Parallel

It also supports more advanced optimisations, especially when combined with NumPy, such as parallel loops if your iterations are independent.

This can be enabled by extending the decorator to `@njit(parallel=True)`.

Additionally loops that are independent should be updated to use `prange`, also imported from `numba`, rather than `range`. This Numba feature is used to tell Numba which loops can be parallelised safely.

**⚠️ Caution when parallelising loops**

If multiple threads access the same variable or array element, and at least one thread modifies it (a **race condition**), the result may be incorrect.

Numba **will not warn you** in this case. Ensure that loop iterations are independent, or you may need to manually rewrite your function using Numba's parallel primitives to handle the race condition safely.

### Extra Packages

You may find that Numba requires additional packages. For example if compiling certain NumPy functions it will require `scipy` to be available to Python. If it's not available, you will receive an error, e.g.

```
ImportError: scipy 0.16+ is required for linear algebra
```

## Example Code

This can be easily demonstrated with 3 versions of the same function below, which includes a basic reduction on the variable `w`.

```py
import numpy as np
from timeit import timeit

# Import the njit decorator
from numba import njit, prange

# Number of points and dimensions
N = 50000
D = 100

# Random 3D points
A = np.random.rand(N, D)
B = np.random.rand(N, D)

# Pure Python version
def python_pairwise(A, B):
    N = A.shape[0]
    out = np.empty(N)
    for i in range(N):
        dist = 0.0
        for j in range(D):
            diff = A[i, j] - B[i, j]
            dist += diff ** 2
        out[i] = np.sqrt(dist)
    return out

# Serial Numba
@njit
def njit_pairwise(A, B):
    N, D = A.shape
    out = np.empty(N)
    for i in range(N):
        dist = 0.0
        for j in range(D):
            diff = A[i, j] - B[i, j]
            dist += diff ** 2
        out[i] = np.sqrt(dist)
    return out

# Parallel Numba
@njit(parallel=True)
def njit_pairwise_parallel(A, B):
    N, D = A.shape
    out = np.empty(N)
    for i in prange(N):  # <-- independent iterations
        dist = 0.0
        for j in range(D):
            diff = A[i, j] - B[i, j]
            dist += diff ** 2
        out[i] = np.sqrt(dist)
    return out

repeats=10
# Benchmark the original Python version
print(f"python: {timeit(lambda: python_pairwise(A, B), number=repeats):.2f}s")

# Benchmark the serial Numba version
print(f"njit_first: {timeit(lambda: njit_pairwise(A, B), number=1):.2f}s")
print(f"njit: {timeit(lambda: njit_pairwise(A, B), number=repeats):.2f}s")

# Benchmark the parallel Numba version
print(f"njit_parallel_first: {timeit(lambda: njit_pairwise_parallel(A, B), number=1):.2f}s")
print(f"njit_parallel: {timeit(lambda: njit_pairwise_parallel(A, B), number=repeats):.2f}s")
```

The `njit` benchmarks are split into a first call, followed by the actual benchmark. This allows the cost of the first-run's compilation to be identified.

* `python` took 21.07s to complete.
* `njit_first` took 0.71s, approximately 0.66ms of which was compilation time.
* `njit` subsequently averaged 0.05s, which is over 400x faster than the original Python.
* `njit_parallel_first` had a similar compile time of 0.61s.
* `njit_parallel` took 0.02s, over 1000x faster!

```output
python: 21.70s
njit_first: 0.71s
njit: 0.05s
njit_parallel_first: 0.63s
njit_parallel: 0.02s
```

## The Technical Detail

CPython is an interpreted language, when you first run a Python script it is typically compiled to Python bytecode (which ends up in `__pycache__`). The CPython interpreter, written in C, then quickly processes the bytecode.

In contrast compiled languages like C, which many high performance Python libraries are written in, compile to machine code which is able to run directly on the processor. This incurs less overhead, as the processor executes the code directly, rather than the processor executing code, to run the code.

Numba's njit effectively attempts to compile decorated Python functions to C, so that they can take advantage of this speedup too!

