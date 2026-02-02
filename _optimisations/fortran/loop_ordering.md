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
At `-O0` optimisation, this will likely run several times faster than the (not
recommended) equivalent row-major version:
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

### Benchmark

Below these examples have been extended into a full benchmark, to demonstrate the impact

<details>
<summary>Benchmark Code</summary>

```fortran
program benchmark_loops
  implicit none
  integer, parameter :: n = 1000
  real, allocatable :: a(:,:,:), b(:,:,:), c(:,:,:)
  real :: t1, t2
  integer :: i
  real :: checksum

  allocate(a(n,n,n), b(n,n,n), c(n,n,n))

  ! Initialize data
  a = 1.0
  b = 2.0
  c = 0.0

  ! Warm-up runs (optional but recommended)
  call col_major_add(a, b, c, n)
  call row_major_add(a, b, c, n)

  ! Benchmark column-major loop order
  call cpu_time(t1)
  call col_major_add(a, b, c, n)
  call cpu_time(t2)
  print *, "Column-major time (s): ", t2 - t1
  checksum = sum(c)
  print *," checksum: ", checksum

  ! Reset result array to avoid reuse artifacts
  c = 0.0

  ! Benchmark row-major loop order
  call cpu_time(t1)
  call row_major_add(a, b, c, n)
  call cpu_time(t2)
  print *, "Row-major time (s):    ", t2 - t1
  checksum = sum(c)
  print *," checksum: ", checksum

contains

  subroutine col_major_add(a, b, c, n)
    integer, intent(in) :: n
    real, intent(in)  :: a(n,n,n), b(n,n,n)
    real, intent(out) :: c(n,n,n)
    integer :: i, j, k

    do k = 1, n
      do j = 1, n
        do i = 1, n
          c(i,j,k) = a(i,j,k) + b(i,j,k)
        end do
      end do
    end do
  end subroutine col_major_add

  subroutine row_major_add(a, b, c, n)
    integer, intent(in) :: n
    real, intent(in)  :: a(n,n,n), b(n,n,n)
    real, intent(out) :: c(n,n,n)
    integer :: i, j, k

    do i = 1, n
      do j = 1, n
        do k = 1, n
          c(i,j,k) = a(i,j,k) + b(i,j,k)
        end do
      end do
    end do
  end subroutine row_major_add

end program benchmark_loops
```

</details>

Compiled with GFortran 11.4.0 and compiler optimisations enabled with `-O3`, column-major accesses were 7x faster!

```output
$ gfortran -O3 benchmark.f90
$ ./a.out
 Column-major time (s):   0.440809250
  checksum:    67108864.0
 Row-major time (s):       3.11280060
  checksum:    67108864.0
```

## Technical Details

The underlying reason that order has such a big impact here is due to how computer memory operates. When a processor loads a variable from RAM into it's caches, it doesn't load an individual variable (likely 4 bytes), it loads a full cache line (likely 64 bytes).

Therefore, variables stored consecutively in memory will be loaded at the same time if they fall within the same cache line. This means the processor does not need to go to RAM to access the variable, which is orders of magnitude higher latency. Due to how Fortran lays out memory, column-major ordering achieves this.

In contrast, if you operate as though memory is row-major, consecutively accessed variables will be stored a long way apart from each other in memory. Hence each individual access will need to load a full cache line from RAM. Due to this high turnover, it's likely by the time a second variable would be accessed from any cache line, that the cache line has been evicted to be replaced with a fresher load.

If you're working with 4-byte types you could be doing 16x the RAM accesses, with 8-byte types it's still 8x!

Furthermore, in order for the compiler to perform vectorisation, it needs to be able to apply vector instructions to a full cache line of variables. If you are not operating consecutively on variables within a cache line, it can be much harder for the compiler to detect that vectorisation is appropriate.
