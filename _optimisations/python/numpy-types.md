---
level: 1
published: true
authors: Robert Chisholm

name: Use numpy.array, rather than Python list, with NumPy functions
language: [Python]
subcategory: [NumPy]
tags: [list, array]
---

NumPy functions are designed to work with NumPy classes (e.g. `numpy.array`), whilst many work with core Python classes (e.g. `list`) this normally incurs an additional cost of NumPy first converting the argument to the corresponding NumPy type.

<!--more-->

If you are passing long `list` this cost is higher, and if you're performing it repeatedly you're incurring that redundant conversion multiple times.

## Example Code

The below example sees an identical `list` and`numpy. array` constructed of 1 million length.

`numpy.random.choice()` is then called to select a random element 100 times.

```sh
import timeit

# Common untimed setup
setup = f"""
N = 1000000
import numpy as np
ls = list(range(1000000))
ar = np.arange(N)
"""

# List version
ls_t = timeit.timeit('np.random.choice(ls)', number=100, setup=setup)
 
# NumPy array version
ar_t = timeit.timeit('np.random.choice(ar)', number=100, setup=setup)

print("np.random.choice(list): %.4fms"%(ls_t))
print("np.random.choice(np.array): %.4fms"%(ar_t))
```

Passing a `numpy.array` was over 4000x faster!

```output
np.random.choice(list): 4.3459ms
np.random.choice(np.array): 0.0010ms
```

So where possible, use a `numpy.array`, even if that means temporarily converting the list yourself before making repeated calls to NumPy functions.

## The Technical Detail

When this issue was [first identified](https://github.com/numpy/numpy/issues/25371), it was shown that `numpy.random.choice()` always calls `np.array(..., copy=False)` when passed a `list` ([source link](https://github.com/numpy/numpy/blob/0be4154648cd6e90aae32246349085f7ca8f2b13/numpy/random/_generator.pyx#L789)).

We can benchmark that too

```sh
import timeit

# Common untimed setup
setup = f"""
N = 1000000
import numpy as np
ls = list(range(1000000))
"""

# List time
ls_t = timeit.timeit('np.random.choice(ls)', number=100, setup=setup)
 
# List copy time
lc_t = timeit.timeit('np.array(ls, copy=False)', number=100, setup=setup)

print("np.random.choice(list): %.2fms"%(ls_t))
print("np.array(list, copy=False): %.2fms"%(lc_t))
```

Which shows that 99% of the runtime is the copy.

```output
np.random.choice(list): 4.37ms
np.array(list, copy=False): 4.32ms
```

NumPy developers confirmed in the [bug report](https://github.com/numpy/numpy/issues/25371 that this performance issue is more widespread, and not something they intend to resolve.

For the example used in this guide, generating a random integer to select the element (e.g. `numpy.random.randint(len(ls))`) would be even faster yet. This is regardless of whether a `list` or `numpy.array` is used, as it avoids some additional branching due to the complex configuration `numpy.random.choice()` permits.