---
level: 1
published: true
authors: Connor Aird

name: Move invariant conditional out of the loop to facilitate vectorisation
language: [C/C++, Fortran]
subcategory: [Core]
tags: [vectorisation]
---

Many Fortran compilers will attempt to automatically vectorise a loop if possible. There are several bad practices which can inhibit this automatic vectorisation.
One such pattern is when a conditional evaluates to the same value for all loop iterations and can be moved outside the loop. In this scenario, not only are we
redundantly evaluating the same conditional but we are also inhibiting automatic vectorisation.

<!--more-->

## Example Code

### C/C++

The following code shows an example of an invariant conditional inside a loop within C/C++

```c++
int example(int *A, int n) {
  int total = 0;

  for (int i = 0; i < n; ++i) {
    if (n < 10) {
      total++;
    }
    A[i] = total;
  }

  return total;
}
```

The loop invariant can be extracted out of the loop by duplicating the loop body and removing the condition. The resulting code is as follows:

```c++
int example(int *A, int n) {
  int total = 0;

  if (n < 10) {
    for (int i = 0; i < n; ++i) {
      A[i] = ++total;
    }
  } else {
    for (int i = 0; i < n; ++i) {
      A[i] = total;
    }
  }

  return total;
}
```

### Fortran

The following code shows an example of an invariant conditional inside a loop within Fortran

```fortran
pure subroutine example(array)
  integer, intent(out) :: array(:)
  integer :: i, total

  total = 0

  do i = 1, size(array, 1)
    if (size(array, 1) < 10) then
      total = total + 1
    end if
    array(i) = total
  end do
end subroutine example
```

The loop invariant can be extracted out of the loop by duplicating the loop body and removing the condition. The resulting code is as follows:

```fortran
pure subroutine example(array)
  integer, intent(out) :: array(:)
  integer :: i, total

  total = 0

  if (size(array, 1) < 10) then
    do i = 1, size(array, 1)
      total = total + 1
      array(i) = total
    end do
  else
    do i = 1, size(array, 1)
      array(i) = total
    end do
  end if
end subroutine example
```

## References

- Examples have been reproduced from the codee open-catalog - [PWR022](https://open-catalog.codee.com/Checks/PWR022/).
