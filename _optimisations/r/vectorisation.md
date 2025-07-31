---
level: 1
published: true
authors: Robert Chisholm

name: Vectorisation
language: [R]
subcategory: [Core]
tags: [array, maths, loops]
---

R has many functions which operate on entire vectors and matrices, which have been implemented in faster C. Vectorisation can perform mathematical operations many times faster than a for loop and even take advantage of conditional logic.

<!--more-->

Note, when R users talk about vectorisation they're typically referring to functions which operate on whole vectors, not advanced SIMD instructions.

## Example Code

To take advantage of vectorisation within R, operations should be performed at an array or matrix level, rather than by looping over the individual elements or rows.

The below example sums the rows within a 1000x1000 matrix to produce an output array with length 1000.

```r
# Allocate an 1000x1000 matrix
mat <- matrix(1:1e6, nrow = 1000)
# Allocate an output array length 1000
x <- integer(length = nrow(mat))
# Calculate the sum of each row into the output array
for (i in 1:nrow(mat)) { 
  x[i] <- sum(mat[i,]) 
}
```

To use vectorisation, it can be rewritten as a single line of code using `rowSums()`.

```r
# Allocate an 1000x1000 matrix
mat <- matrix(1:1e6, nrow = 1000)
tic()
# Call the rowSums() method
x <- rowSums(mat)
toc()

```

In this example vectorisation also reduces the runtime from 0.05 to 0.02 seconds.

Most R functions support vectorisation, so it's important to try and use purpose built functions to process your arrays and matrices, rather than manually looping.

Not all of your R code will be this simple though, maybe you have conditional logic which restricts which elements the operation should be performed on.

```r
# Allocate an array of 1 million elements
x <- 1:1e6
# Divide even elements by 2, multiply odd elements by 3

for (i in x) {
  if (i %% 2 == 0) {
    i <- i / 2         # even: divide by 2
  } else {
    i <- i * 3         # odd: multiply by 3
  }
}
```

The above example can be written using `ifelse()` as

```r
# Allocate an array of 1 million elements
x <- 1:1e6
# Divide even elements by 2, multiply odd elements by 3
result <- ifelse(x %% 2 == 0,         # condition: even numbers
                 x / 2,               # if TRUE: divide even numbers by 2
                 x * 3)               # if FALSE: multiply odd numbers by 3
```

The `ifelse()` approach reduced the runtime from 0.35 seconds to 0.06 seconds.

Using vectorisation is both more concise and more performant.

You might be tempted to use `sapply()` or `vapply()`, which allow you to apply a transformation function to each element within an array, but these were slower than the original loop in testing.

## The Technical Detail

Many of R's mathematical vector functions rely on the scientific libraries BLAS and LAPACK, which provide efficient implementations of the operations. These are faster when executed in a single call to C, rather than an R loop over a vector which makes many individual calls.

BLAS and LAPACK can take advantage of even faster SIMD (Single Instruction, Multiple Data) instructions to operate on several values in parallel which are packed into a register. However, most R installations are provided with a reference implementation of these libraries which is unlikely to take advantage of these. It's currently non-trivial to install R with an accelerated version of BLAS, but it's likely your will find one [setup on HPC systems](https://rse.shef.ac.uk/blog/intel-r-iceberg/).
