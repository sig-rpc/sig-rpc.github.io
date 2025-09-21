---
level: 1
published: true
authors: Joe Wallwork

name: Reshape
language: [Fortran]
subcategory: [Core]
tags: [reshape]
---

It's often required to reshape arrays used in Fortran code. This can be achieved
in several ways, the most naive of which is to use hand-coded `do` loops. This
is not recommended as it is error-prone and verbose. A better approach is to use
the intrinsic `reshape` function, which is concise and clear.

The most efficient way to reshape arrays is to use pointers. This is a more
advanced approach and care must be taken to ensure that (a) the original array
is not deallocated while the pointer is still in use and (b) you are aware that
modifications to the "reshaped" array will modify the original array and vice
versa because they share the same memory.

<!--more-->

The `pack` and `unpack` intrinsics can also be used to reshape arrays but this
is not their intended purpose and they are less efficient than `reshape` at
achieving this.

It is also possible to do *implicit* reshaping of arrays by passing them to
procedures with different interface shapes. This is generally not recommended as
it can lead to confusing code and requires compiler flags such as
`-fallow-argument-mismatch` in the case of `gfortran`.

## Example Code

The 1D to 2D reshape coded by hand in
```fortran
integer, parameter :: m = 3, n = 4
integer, parameter :: mn = m * n
real , dimension(mn) :: arr1d
real , dimension(m, n) :: arr2d

! Initialisation of arr1d omitted

! Hand-coded reshape
do j = 1, n
  do i = 1, m
    arr2d(i, j) = arr1d((i - 1) * n + j)
  end do
end do
```
can be implemented using `reshape` as simply
```fortran
arr2d = reshape(arr1d, shape(arr2d))
```

To implement this using pointers, use the `c_f_pointer` subroutine and `c_loc`
function from the `iso_c_binding` module (intrinsic since Fortran 2003). The
`c_f_pointer` subroutine associates a C pointer with a Fortran pointer, which
allows you to "reshape" the array simply by aliasing (without copying data).
```fortran
use iso_c_binding, only: c_f_pointer, c_loc

integer, parameter :: m = 3, n = 4
integer, parameter :: mn = m * n
real, target, dimension(mn) :: arr1d
real, pointer :: arr2d(:,:)

! Initialisation of arr1d omitted

! Associate arr2d with arr1d, reshaping it to (m, n)
call c_f_pointer(c_loc(arr1d), arr2d, [m, n])

! Use arr2d as needed

! Don't forget to nullify the pointer when it's no longer needed
nullify(arr2d)
```
