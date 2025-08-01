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

This took 4.42 seconds (141.91 seconds for 100,000 rows!).

But if we initialise the data-frame with the correct number of rows, and instead update the elements in-place.

```r
# Allocate a data.frame with 3 columns and preallocate 10,000 rows.
df <- data.frame(x = numeric(10000), y = numeric(10000), z = numeric(10000))
# Append 10,000 rows
for (i in 1:10000) {
  df$x[i] <- i
  df$y[i] <- i * 2
  df$z[i] <- i * 3
}
```

This now executes in 0.81 seconds (26.56 seconds for 100,000 rows).

We can however go one further, if we instead use `data.table` and take advantage of [vectorisation](vectorisation).

```r
# Import the data.table library
library(data.table)
df <- data.table(x = numeric(10000), y = numeric(10000), z = numeric(10000))
rows <- 1:10000
df[rows, `:=`(
  x = rows,
  y = rows * 2,
  z = rows * 3
)]
```

This now executes in 0.05 seconds (and doesn't get slower for 100,000 elements)

If you need to convert it back to a `data.frame`, you can call `as.data.frame()` passing your `data.table` as a parameter. 

## The Technical Detail

R data-frames are column major, and many operations force the columns to be re-allocated. In particular, if you add a row each column of the data-table will be re-allocated.

`data.table` is almost always equal if not faster in performance to the core `data.frame`, so there's little reason not to use it.
