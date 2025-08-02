---
level: 1
published: true
authors: Robert Chisholm

name: Don't Grow data.frame(s)
language: [R]
subcategory: [Core]
tags: [data.frame, vector, memory]
---

When you grow a `data.frame` by one, for example using `rbind()` or `cbind()` to add a row or column, R will allocate a new `data.frame` one larger and copy across all the old data. If you're doing this regularly, that's alot of redundant expensive memory movement. Either pre-allocate your `data.frame` to the correct size at the start, or use a `data.table` instead which provides faster growth (and many other performance improvements).

<!--more-->

Prior to R 3.4, vectors suffered from the same overhead during growth. However, the newer implementation ensures this overhead of redundant copies is not incurred most of the time. It may still occur when there are multiple references to a growing vector, so beware.

## Example Code

It might feel natural to grow a `data.frame` row-by-row as you collect data.

```r
# Allocate a data.frame with 3 columns
df <- data.frame(x = numeric(), y = numeric(), z = numeric())
# Append 10,000 rows
for (i in 1:10000) {
  new_row <- data.frame(x = i, y = i * 2, z = i * 3)
  df <- rbind(df, new_row)
}
```
But this took 3.27 seconds (103.92 seconds for 100,000 rows!).

Prefer using a `data.table` instead.

```r
# Import the data.table library
library(data.table)
# Allocate a data.table with 3 columns
df <- data.table(x = numeric(1e6), y = numeric(1e6), z = numeric(1e6))
# Update 10,000 rows
for (i in 1:10000) {
  df[[i]] <- data.table(x = i, y = i * 2, z = i * 3)
}
toc()

```

We've updated this by importing `data.table` and updating the type of `df`, otherwise the syntax is identical.

But it finishes executing in 0.04 seconds (and didn't get slower for 100,000 rows!).

If you need to convert it back to a `data.frame`, you can call `as.data.frame()` passing your `data.table` as a parameter. 

## The Technical Detail

R data-frames are column major, and many operations force the columns to be re-allocated. In particular, if you add a row each column of the data-table will be re-allocated.

`data.table` is almost always equal if not faster in performance to the core `data.frame`, so there's little reason not to use it.

*If you know more about why the performance of `data.frame` suffers so badly, please contribute. In testing pre-allocating a `data.frame` came out slower than growing, why?*