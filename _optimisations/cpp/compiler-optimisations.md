---
level: 1
published: true
authors: Robert Chisholm

name: Compiler Optimisations (Release Builds)
language: [C/C++, Fortran]
subcategory: [Core]
tags: [compiler]
---

Modern compilers support many automatic low-level optimisations which can improve the performance of the compiled code. The actual performance gain depends on your specific code, but it is not uncommon to see 5–10x speedups, and in some cases, 50x or more, especially for compute-intensive tasks.

<!--more-->

Compiler optimisations are not enabled by default because they make debugging more difficult. Optimised builds can:

* Reorder or eliminate code, making it harder to set breakpoints or inspect variables
* Inline functions, obscuring the call stack
* Remove or coalesce variables that appear unused

As a result, developers typically use unoptimised builds (debug) during development and optimised builds (release) for production.

## Enabling Optimisations

If you wish to enable compiler optimisations (or check whether they are enabled) the guidance differs slightly between compilers.

**GCC / Clang**

The main optimisation flags provided by GCC and Clang are:

* `-O0`: No optimisation (default)
* `-O1`, `-O2`, `-O3`: Increasing levels of optimisation
* `-Ofast`: Aggressive optimisations, may break strict standards compliance
* `-Os`: Optimize for size

You should typically ensure that `-O2` is passed to the compiler (either on the command-line or within the makefile/project) when compiling for release, to take advantage of compiler optimisations. If you're not sure whether they're being used, look out for them in the compile log.

`-O3` and `-Ofast` can provide mixed results, as they use the most aggressive optimisations which can have inadvertent effects such as greatly increasing the binary size for limited to no further impact to performance.

**CMake**

CMake supports Debug and Release build configurations via the `CMAKE_BUILD_TYPE` argument at configure time.

```sh
cmake -DCMAKE_BUILD_TYPE=Release .
cmake --build .
```

If the CMake project is correctly structured (it may be worth reviewing the compilation log to make sure you can see an optimisation flag), passing `Release` will enable compiler optimisations . It's up to the project maintainer in this scenario, whether the default configuration corresponds to Debug, Release or something else entirely.


**Visual Studio (MSVC)**

The default project structure within visual studio will contain both a Debug and a Release build configuration, the latter will have compiler optimisations enabled and should be used for production builds.

To enable compiler optimisations manually, navigate to *Project Properties* > *C/C++* > *Optimization* and set this value to `/O2` (Maxmimise speed) or `/Ox` (Full optimisation).

## The Technical Detail

There are many different optimisations which can be performed by a compiler, GCC [lists most of their compiler optimisations](https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html) and allows them to be individually toggled. Passing `-Q --help=optimizers` to `gcc`, `g++`, or `gfortran` will list the exact set of optimisations enabled by the chosen optimisation level.

