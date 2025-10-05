---
level: 1
published: true
authors: Mike Laverick, Sam Cunliffe

name: Searching with a for-loop
language: [Python]
subcategory: [Core]
tags: [loop, search, operator]
---

Searching for an element in a sequence (e.g. a list) with a for-loop and an equality check is very natural, but Python has a built-in `in` keyword which should be used whenever possible.

<!--more-->

## Example Code

In this toy example, we generate a large number of 5-letter words using any ASCII character. We search for the presence of capital Z in each word in the list.

```python
from timeit import timeit
import random
import string

N = 50000  # Number of elements in list
SEARCH_LETTER = "Z" # Searching for capital Z


def generate_words():
    random.seed(12)  # Ensure every list is the same
    return [''.join(random.choices(string.ascii_letters, k=5)) for _ in range(N)]


def for_loop_search():
    list_of_words = generate_words()
    for word in list_of_words:
        for letter in word:
            if letter == SEARCH_LETTER:
                break


def in_search():
    list_of_words = generate_words()
    for word in list_of_words:
        if SEARCH_LETTER in word:
            pass


repeats = 1000
start_time = timeit(generate_words, number=repeats)
print(f"for-loop search: {timeit(for_loop_search, number=repeats) - start_time:.2f}ms")
print(f"'in' keyword search: {timeit(in_search, number=repeats) - start_time:.2f}ms")
```

In testing, this produced the following results

```
for-loop search: 4.01ms
'in' keyword search: 0.88ms
```

The in keyword performed 4.5x faster than the for-loop, the exact speedup is likely to vary with the length of the sequence being searched.
