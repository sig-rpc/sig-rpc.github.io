---
level: 1
published: true
authors: Robert Chisholm

name: Set 
language: [Python]
subcategory: [Core]
tags: [list, set, duplicate, unique]
---

Similar to the mathematical concept of a set, Python (and most other languages) provides a [data structure `set`](https://docs.python.org/3/tutorial/datastructures.html#sets) which is an unordered collection of unique values. Using a `set` is the fastest way to detect unique items (e.g. `if a in my_set`) or remove duplicates (e.g. `[x for x in set(my_list)]`).

<!--more-->

## Example Code

Rather than using a `list` to build a unique collection of items

```py
import random
# Create a list of 1000 random numbers in the range [0, 2000)
list_out = []
while len(list_out) < 1000:
    t = random.randint(0, 2000)
    if not t in list_out:
        list_out.append(t)
```

Use a `set`

```py
import random
# Create a set of 1000 random numbers in the range [0, 2000)
set_out = []
while len(set_out) < 1000:
    t = random.randint(0, 2000)
    set_out.add(t)
```

Depending on the size of the collection, the order of items, and the proportion of duplicates, using a `set` could be thousands of times faster than using a `list`.

<!-- Todo, can this be made into a WebASM example? -->
<details markdown="block">
<summary>Example Benchmark</summary>
The below code provides a simple benchmark of removing duplicates from a list of 25000 random integers using a `list` or `set`.

```py
import random
from timeit import timeit

# A simple method to generate us a consistent input list
def generateInputs(N = 25000):
    random.seed(12)  # Ensure every list is the same 
    return [random.randint(0,int(N/2)) for i in range(N)]

# Pass the list directly to the set constructor
def uniqueSet():
    ls_in = generateInputs()
    set_out = set(ls_in)

# Iterate the list, adding each item to the set individually
def uniqueSetAdd():
    ls_in = generateInputs()
    set_out = set()
    for i in ls_in:
        set_out.add(i)

# Iterate the list, adding each unique item to the new list individually
def uniqueList():
    ls_in = generateInputs()
    ls_out = []
    for i in ls_in:
        if not i in ls_out:
            ls_out.append(i)

# Sort the input list, add each item if it does not match the last item of the new list
def uniqueListSort():
    ls_in = generateInputs()
    ls_in.sort()
    ls_out = [ls_in[0]]
    for i in ls_in:
        if ls_out[-1] != i:
            ls_out.append(i)
            
repeats = 1000
gen_time = timeit(generateInputs, number=repeats)
print(f"uniqueSet: {timeit(uniqueSet, number=repeats)-gen_time:.2f}ms")
print(f"uniqueSetAdd: {timeit(uniqueSetAdd, number=repeats)-gen_time:.2f}ms")
print(f"uniqueList: {timeit(uniqueList, number=repeats)-gen_time:.2f}ms")
print(f"uniqueListSort: {timeit(uniqueListSort, number=repeats)-gen_time:.2f}ms"
```

In testing this produced the following results

```
uniqueSet: 0.30ms
uniqueSetAdd: 0.81ms
uniqueList: 660.71ms
uniqueListSort: 2.67ms
```

Using the constructor for `set` was over 2000x times faster than iterating the unsorted list.

</details>

## The Technical Detail

Set data structures are similar to dictionaries, but without the values. Internally they are typically implemented with hashing or tree data-structures. These are highly optimal for direct access to items, requiring less items to be checked to test for existence.

In contrast, performing a search through an unsorted list will require all items to be checked in the worst case whereby the item is not found. Approaches such as sorting the list and using a binary search can greatly improve performance, however in most cases using a `set` will be preferable.
