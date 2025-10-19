---
level: 2
published: true
authors: Rastislav Turanyi, Robert Chisholm

name: Use BLAS/LAPACK Instead of Reimplementing Maths Functions
language: [C/C++, Fortran]
subcategory: [Maths]
tags: [maths, hpc, linear algebra, matrix]
---

[BLAS](https://www.netlib.org/blas/) (Basic Linear Algebra Subprograms) and [LAPACK](https://www.netlib.org/lapack/) (Linear Algebra PACKage) are widely used open-source libraries for doing maths, especially matrix operations. They are highly optimised, often for specific CPU architectures by the vendors (Intel, AMD, etc.) themselves, and can be found on all modern HPC systems. Therefore, they are significantly faster than anything we can write ourselves, especially for larger matrices, and should be used in most circumstances.

<!--more-->

The BLAS and LAPACK standards are originally defined using Fortran, so using them is a little bit cumbersome:

1. The function signature for any BLAS/LAPACK function to be used has to be first declared.
2. If using C/C++, the function name must have an underscore, e.g. `dgemm` is used as `dgemm_`.
3. All arguments must be passed by reference.
4. Matrices must be contiguous arrays in column-major order (more on this below).

Other than that, using them is very straightforward — all we need to do is call a function in our code, and then link against BLAS when compiling our code, e.g.:

```
gcc example.c -o example -lopenblas
```

or in CMake:

```
find_package(BLAS REQUIRED)

add_executable(example example.c)
target_link_libraries(example ${BLAS_LIBRARIES})
```

**IMPORTANT**: When using BLAS/LAPACK, make sure you don't link against the reference BLAS (this is the reference implementation and has not been optimised). A good shorthand is to use OpenBLAS, but depending on your CPU etc., other implementations may be faster. Check which implementations are available on your HPC system and ask your local friendly HPC administrators if unsure.


## Example

This is a small benchmark comparing a hand coded matrix multiplication versus BLAS's implementation:

```cpp
#include <cstdlib>
#include <cstdio>
#include <chrono>

/**
 * Crude random matrix generator
 */
double* generate_matrix(int m, int n) {
  double* a = (double*)malloc(m * n * sizeof(double));
  for (int i = 0; i < m * n; ++i)
    a[i] = (double)rand() / RAND_MAX;
  return a;
}

/**
 * Manually implemented matrix multiplication
 */
void multiply(const double *matrix1, int m1, int n1, const double *matrix2, int m2, int n2, double *result) {
  for (int i = 0; i < m1; i++) {
    for (int j = 0; j < n2; j++) {
      result[i + j * m1] - 0.0;

      for (int k = 0; k < n1; k++) {
        result[i + j * m1] += matrix1[i + k * m1] * matrix2[k + j * m2];
      }
    }
  }
}

/**
* BLAS's matrix multiplication prototype
*/
extern "C" {
void dgemm_(const char*, const char*, const int*, const int*, const int*, const double*, const double*, const int*, const double*, const int*, const double*, double*, const int*);
}

int main() {
  const int m = 1000, n = 1000;
  const int REPEATS = 10;

  double *matrix1 = generate_matrix(m, n);
  double *matrix2 = generate_matrix(m, n);
  double *result_m = (double*)malloc(m * n * sizeof(double));
  double *result_b = (double*)malloc(m * n * sizeof(double));

  // Benchmark manual version
  auto start_m = std::chrono::high_resolution_clock::now();
  for (int i = 0; i < REPEATS; ++i)
    multiply(matrix1, m, n, matrix2, m, n, result_m);
  auto end_m = std::chrono::high_resolution_clock::now();
  
  std::chrono::duration<double> elapsed_m = end_m - start_m;
  printf("Manual took an average of %f seconds.\n", elapsed_m.count()/REPEATS);
  
  // Benchmark BLAS version
  const double one = 1.0, zero = 0.0;
  auto start_b = std::chrono::high_resolution_clock::now();
  for (int i = 0; i < REPEATS; ++i)
    dgemm_("N", "N", &m, &n, &n, &one, matrix1, &m, matrix2, &m, &zero, result_b, &m);
  auto end_b = std::chrono::high_resolution_clock::now();
  
  std::chrono::duration<double> elapsed_b = end_b - start_b;
  printf("BLAS took an average of %f seconds.\n", elapsed_b.count()/REPEATS);

  free(matrix1); free(matrix2); free(result_m); free(result_b);
}
```

Tested on local HPC, with 16-cores, using OpenBLAS:

```sh
module load OpenBLAS
g++ bench.cpp -O3 -o bench -lopenblas
./bench
```

Output:

```output
Manual took an average of 1.183420 seconds.
BLAS took an average of 0.002624 seconds.
```

That's a 450x speed-up!

## CBLAS

The original BLAS interface expects column-major matrices, [CBLAS](https://www.netlib.org/lapack/explore-html/de/da0/cblas_8h.html) can be used under C/C++ to process row-major matrices which may even perform faster due to improved memory access patterns.

Include `<cblas.h>` and prepend `cblas_` to the method name. Note that `char` arguments have been replaced with enums and matrix functions have the additional layout argument, so the exact parameters passed to the function are likely to require updating.

```c++
/**
 * CBLAS include
 */
#include <cblas.h>

int main() {
...
  // Benchmark CBLAS version
  auto start_cb = std::chrono::high_resolution_clock::now();
  for (int i = 0; i < REPEATS; ++i)
    cblas_dgemm(CblasRowMajor, CblasNoTrans, CblasNoTrans, m, n, n, 1.0, matrix1, n, matrix2, n, 0.0, result_cb, n);
  auto end_cb = std::chrono::high_resolution_clock::now();
  
  std::chrono::duration<double> elapsed_cb = end_cb - start_cb;
  printf("CBLAS took an average of %f seconds.\n", elapsed_cb.count()/REPEATS)
}

```
Which reports a further 20% speedup!

```output
CBLAS took an average of 0.002022 seconds.
```


## GPU

Even better, modern BLAS and LAPACK implementations are now available for GPUs, allowing us to take advantage of these accelerators with minimal effort (though unfortunately the interface is often different):

- NVIDIA has [cuBLAS](https://developer.nvidia.com/cublas#:~:text=The%20cuBLAS%20library%20is%20highly,Datacenter%20GPUs%20for%20various%20precisions) (though not LAPACK implementation, unfortunately)
- AMD has [rocblas/hipblas](https://github.com/ROCm/rocm-libraries)
- There are also cross-platform (work on both NVIDIA and AMD hardware) implementations like [magmaBLAS](https://github.com/icl-utk-edu/magma)

## Technical Explanation

[There is a surprising amount of low-level optimisation possible when writing linear algebra functions](https://www.cs.utexas.edu/~rvdg/tmp/TSoPMC.pdf) - writing a simple double or triple loop to perform the operation is only the beginning, and the compiler can only do so much. One of the basic optimisations is computing the operation in blocks, but the block size needs to be optimised to the cache sizes available on a specific CPU. However, the HPC implementations of these libraries go way further than that, even going as far as writing the innermost computations directly in assembly. Furthermore, many HPC implementations are also parallelised with OpenMP, so using them gives us some parallelism "for free".

