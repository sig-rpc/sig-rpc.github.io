---
level: 1
published: true
authors: Joe Wallwork

name: Loop ordering
language: [Fortran, MATLAB]
subcategory: [Core]
tags: [loop, array]
---

In Fortran and MATLAB, arrays are stored in column-major order, meaning that the
leftmost index varies most quickly. This is different from many other popular
scientific programming languages such as C, C++, and Fortran, which use a
row-major ordering. The implication for writing loops is that the ordering of
loops is important - it is always best to order loops by which indices vary
quickest.

<!--more-->

## Example Code

Consider the following Fortran example involving a loop over a 3D array, which
uses the recommended column-major ordering:
```fortran
integer, parameter :: n = 500
real, dimension(n, n, n) :: a, b, c
integer :: i, j, k

! Initialisation of a and b omitted

! Hand-coded c = a + b
do k = 1, n
  do j = 1, n
    do i = 1, n
      c(i,j,k) = a(i,j,k) + b(i,j,k)
    end do
  end do
end do
```
At `-O0` optimisation, this will likely run four or more times faster than the
equivalent row-major version:
```fortran
integer, parameter :: n = 500
real, dimension(n, n, n) :: a, b, c
integer :: i, j, k

! Initialisation of a and b omitted

! Hand-coded c = a + b
do i = 1, n
  do j = 1, n
    do k = 1, n
      c(i,j,k) = a(i,j,k) + b(i,j,k)
    end do
  end do
end do
```
