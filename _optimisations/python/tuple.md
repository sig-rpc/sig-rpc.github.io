---
level: 3
published: true
authors: Robert Chisholm

name: Tuple 
language: [Python]
subcategory: [Core]
tags: [list, tuple]
---

In addition to lists, Python has the concept of [tuples](https://docs.python.org/3/tutorial/datastructures.html#tuples-and-sequences). These are immutable static arrays, they cannot be resized, nor can their elements be changed. Their potential use-cases are greatly reduced due to these two limitations, as they are only suitable for groups of immutable properties.

Tuples will typically allocate several times faster than lists with equal contents, therefore they are an ideal replacement if immutable lists are being created thousands of times (tuples are unlikely to make a huge difference to any individual list allocation).

<!--more-->

Python caches a large number of short (1-20 element) tuples, this greatly reduces the cost of creating and destroying them during execution compared to lists.

A tuple is constructed with `( )`, in contrast to `[ ]` used by a list, they can also be constructed with list comprehension mechanics. They can still be joined with the `+` operator, similar to appending lists, however the result is always a newly allocated tuple.

## Example Code

This can be easily demonstrated with Python's `timeit` module in your console.

```sh
# List of length 6
>python -m timeit "li = [0,1,2,3,4,5]"
5000000 loops, best of 5: 69.4 nsec per loop

# Tuple of length 6
>python -m timeit "tu = (0,1,2,3,4,5)"
20000000 loops, best of 5: 17.4 nsec per loop

# List of length 16
>python -m timeit "li = [0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15]"
5000000 loops, best of 5: 90.4 nsec per loop

# Tuple of length 16
>python -m timeit "tu = (0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15)"
20000000 loops, best of 5: 17.6 nsec per loop
```

It takes 4-5x as long to allocate a list than a tuple of equal length. This gap grows with the length, the tuple allocation cost remains roughly static whereas the cost of allocating the list grows slightly.

```sh
# List length 2000 via comprehension
>python -m timeit "li = [i for i in range(2000)]"
5000 loops, best of 5: 66 usec per loop

# Tuple length 2000 via comprehension
>python -m timeit "tu = (i for i in range(2000))"
500000 loops, best of 5: 728 nsec per loop
```

In this larger example using comprehension syntax, the tuple constructs 90x faster.

## The Technical Detail

Tuples are a simpler data-structure than lists, this likely means that in addition to Python pre-caching their storage, there is less internal meta-data to setup during initialisation.

*If you have time to investigate this further by checking the CPython source, please [let us know](https://github.com/sig-rpc/sig-rpc.github.io/issues/new?template=improve_optimisation.yml&title=%5BFix%5D%3A%20Improve%20tuple%20technical%20detail)!*


