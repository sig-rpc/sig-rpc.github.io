---
level: 2
published: true
authors: Robert Chisholm

name: timeit
language: [Python]
style: [Benchmarking]
website: https://docs.python.org/3/library/timeit.html
---

[`timeit`](https://docs.python.org/3/library/timeit.html) is a Python package which is convenient for benchmarking functions and other small snippets of code. As it's core Python, it doesn't require any additional installation of packages to use.

<!--more-->

Due to it's simplicity, it's really convenient for quickly testing the speed of small code snippets. We've used it widely in examples throughout this website!

## QuickStart

It can be used either via the command line, to benchmark short snippets of Python, or within Python code to benchmark both snippets of code and entire functions.

### Command Line

On the command-line a snippet of Python code can be passed directly to `timeit` with `python -m`.
On some systems, you may need to replace `python` with `python3` if Python is called differently.

```sh
python -m timeit "'-'.join(str(n) for n in range(100))"
```

The example above, taken from timeit's documentation, runs the passed code repeatedly to output a small benchmark report.
The number of repetitions (loops) is decided automatically, usually quicker snippets will have more.

```
20000 loops, best of 5: 13.1 usec per loop
```

### Inline Python

First you should import the `timeit` function in your code.

```py
from timeit import timeit
```

Then call `timeit()`, it takes several arguments allowing you to pass independent setup code which is not timed, and to request how many repeats.

The most important arguments (and their defaults) are highlighted below.

```py
timeit.timeit(stmt='pass', setup='pass', number=1000000)
```

* `stmt`: This is the code to be benchmarked. You can pass either: a string containing Python, a function, or a lambda containing a function call.
* `setup`: This optional argument is passed code used to initialise the benchmarking context. This could be allocating and filling a data-structure or similar. This will only be called once, before the benchmark loop.
* `number`: The number of times to run `stmt`. This defaults to 1 million, so for expensive code you may want to pass a smaller number.

`timeit()` then returns the number of seconds it took to run the full loop (not the average time, so divide by `number` if necessary).

**Full Example**

You can find many examples across this knowledge-base, the one below is based on one from the Python optimisation page [Numba](/optimisation/python/numba).

```py
import numpy as np
from timeit import timeit

# Number of points and dimensions
N = 5000
D = 10

# Random 3D points
A = np.random.rand(N, D)
B = np.random.rand(N, D)

# Pure Python version
def pairwise(A, B):
    N = A.shape[0]
    out = np.empty(N)
    for i in range(N):
        dist = 0.0
        for j in range(D):
            diff = A[i, j] - B[i, j]
            dist += diff ** 2
        out[i] = np.sqrt(dist)
    return out

# Benchmark the original Python version
repeats=100
pairwise_seconds = timeit(lambda: pairwise(A, B), number=repeats)
# Output the time
print(f"{repeats} x pairwise(A,B) took {pairwise_seconds:.2f} seconds")
```

In the call to `timeit()` a lambda function to `stmt` which calls `python_pairwise()` with the variables `A` and `B`.

This is followed by a statement to print the time it took.
It's not always necessary to care about it being "seconds", if you're testing multiple tweaks to a function all you probably care about is which is faster.

### Jupyter Notebooks

If you're more familiar with writing Python inside Jupyter notebooks you can still use `timeit` directly from inside notebooks using the notebooks "magic" prefix (`%`) and it will automatically call `timeit` for you.

You can either call `%timeit` inline to profile code on the same line, using functions and variables defined earlier in the notebook (similar to the lambda function demonstrated above).

```py
%timeit pairwise(A, B)
```

Or, you can create a `%%timeit` cell, to profile the python executed within it.

Code on the first line is called once as the setup, and remaining lines of the cell are what will be benchmarked.


```py
%%timeit A=np.random.rand(N, D);B=np.random.rand(N, D)

pairwise(A, B)
```
In both cases this will then output something similar to the command-line interface:

```output
73.5 ms ± 3.13 ms per loop (mean ± std. dev. of 7 runs, 10 loops each)
```

Additional arguments can be passed to both notebook versions, these allow you to control the number of loops and more. You can read about this in the [official notebook documentation](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-timeit).

## Limitations

`timeit` isn't quite a profiler, it will only give you a macroscopic view of the code it benchmarks. If you're looking to find where time is being spent in a large project, consider using a [function-level profiling tool](/profilers/python/?filter=Function-Level).
