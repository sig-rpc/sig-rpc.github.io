---
level: 2
published: true
authors: Donal Fellows, Robert Chisholm
name: Generator Functions 
language: [Python]
subcategory: [Core]
tags: [list, tuple]
---

Using similar notation to [list comprehension](), Python offers [generator functions](https://wiki.python.org/moin/Generators (also called generators). Unlike list comprehension however, generator functions generate the defined sequence as they are accessed. This can be faster if generating a sequence that will only be iterated once.

<!--more-->

Technically `range()` is a generator function!

## Example Code

In this toy example, we perform some arbitrary maths on a sequence that we generate:

```sh
from timeit import timeit

N = 1000000  # Number of elements in list

def list_comprehension():
    abc = [x * 3 for x in range(N)]
    y = 7
    for x in abc:
        y = y * x if x > 173 else y / (x+1)

def generator_function():
    abc = (x * 3 for x in range(N))
    y = 7
    for x in abc:
        y = y * x if x > 173 else y / (x+1)

def range_generator():
    for i in range(N):
        x = i * 3
        y = y * x if x > 173 else y / (x+1)


repeats = 100
print(f"list_comprehension: {timeit(list_comprehension, number=repeats):.2f}ms")
print(f"generator_function: {timeit(generator_function, number=repeats):.2f}ms")
print(f"range_generator: {timeit(range_generator, number=repeats):.2f}ms")
```

Which returned:

```output
list_comprehension: 11.43ms
generator_function: 10.94ms
range_generator: 9.34ms
```

`range()` is a generator function!


## The Technical Detail

Tuples are a simpler data-structure than lists, this likely means that in addition to Python pre-caching their storage, there is less internal meta-data to setup during initialisation.

*If you have time to investigate this further by checking the CPython source, please [let us know](https://github.com/sig-rpc/sig-rpc.github.io/issues/new?template=improve_optimisation.yml&title=%5BFix%5D%3A%20Improve%20tuple%20technical%20detail)!*


