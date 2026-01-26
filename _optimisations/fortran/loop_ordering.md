---
level: 1
published: true
authors: Joe Wallwork

name: Loop ordering
language: [Fortran, MATLAB, R]
subcategory: [Core]
tags: [loop, array]
---

In Fortran, MATLAB, and R, arrays are stored in column-major order, meaning that
the entries from the left-most index are stored contiguously in memory. This is
often referred to as the left-most index _varying quickest_. This is different
from many other popular scientific programming languages such as C, C++, and
Python, which use a row-major ordering. The implication for writing loops is
that it is always best to order loops by which indices vary quickest. In this
way, the loop nest will traverse contiguous memory, which can be done
efficiently.

<!--more-->

## Example Code

Consider the following Fortran example involving a loop over a 3D array, which
uses the recommended column-major ordering:
```fortran
integer, parameter :: n = 500
real, dimension(n, n, n) :: a, b, c
integer :: i, j, k

! Initialisation of a and b omitted

! Hand-coded c = a + b using column-major
do k = 1, n
  do j = 1, n
    do i = 1, n
      c(i,j,k) = a(i,j,k) + b(i,j,k)
    end do
  end do
end do
```
At `-O0` optimisation, this will likely run four or more times faster than the
(not recommended) equivalent row-major version:
```fortran
integer, parameter :: n = 500
real, dimension(n, n, n) :: a, b, c
integer :: i, j, k

! Initialisation of a and b omitted

! Hand-coded c = a + b using row-major
do i = 1, n
  do j = 1, n
    do k = 1, n
      c(i,j,k) = a(i,j,k) + b(i,j,k)
    end do
  end do
end do
```
At higher optimisation levels, simple loops such as this would likely be
optimised out by the compiler. However, this would not necessarily be the case
for more complex loop structures.

A further comment on the simple loops above is that the same could be achieved
with an array operation, which has a more compact notation:
```fortran
integer, parameter :: n = 500
real, dimension(n, n, n) :: a, b, c

! Initialisation of a and b omitted

! Compute c = a + b with an array operation
c(:,:,:) = a + b
```
