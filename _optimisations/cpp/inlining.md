---
level: 1
published: true
authors: Robert Chisholm

name: Inlining
language: [C/C++]
subcategory: [Core]
tags: [function, small, inline]
---

Calling a function has a small overhead, for small (e.g. mathematical) functions this can be high relative to the cost of actually executing the function. By moving regularly called small functions into headers and marking them as `inline`, the compiler will inline them where appropriate when compiler optimisations are enabled, removing the call overhead.

<!--more-->

If when function-level profiling your code you see a function listed, that function has not been inlined. If you have small/cheap  functions (e.g. 1-3 lines of maths) which are called frequently during execution and they appear among the functions using the most CPU time during execution. Consider making them inline, this should reduce the cost of calling them.

The implementation of an inlined function must be visible to the compiler when the function is being called in order for it to be inlined. This typically involves moving the implementation of the inlined function into it's corresponding header file, either within the class definition or after it. Both the function prototype and implementation should be marked `inline`.

The `inline` keyword does not guarantee a compiler will inline calls to the function, compilers may use heuristics to decide when inlining is not appropriate.

The cost of a function call is relatively small, so inlining is only effective for small functions, where the cost of calling the function is a large proportion of it's total cost. The performance impact will only typically be visible over the inlining of many thousands of function calls.

## Example

A project implemented a bespoke small vector class that was used heavily during it's simulations.

**`Vec3.h`**
```cpp
#ifndef VEC3_H__
#define VEC3_H__
/**
 * Three dimensional, double type vector
 */
class Vec3 {
 public:
    // Storage
    double x,y,z;
    // Constructor
    Vec3(const double &a = 0, const double &b = 0, const double &c = 0){
        x=a;
        y=b;
        z=c;
    }
    // Arithmetic operator prototypes
    Vec3 operator+(const Vec3 &vec) const;
    Vec3& operator+=(const Vec3 &vec);
    // etc...
};
#endif // VEC3_H__
```

**`Vec3.cpp`**
```cpp
// Arithmetic operator implementation
Vec3 Vec3::operator+(const Vec3 &vec) const {
    return Vec3(
        x + vec.x,
        y + vec.y,
        z + vec.z);
}

tVect& tVect::operator+=(const Vec3 &vec) {
    x += vec.x;
    y += vec.y;
    z += vec.z;
    return *this;
}
// etc...
```

When this project was profiled with [gProf](/profiler/cpp/gprof), the two most expensive functions were `Vec3::operator+()` and `Vec3::operator*()`, combined they were over 25% of the cumulative runtime, being called around 100 billion times each during the profile.

In response to this they were inlined by adding `inline` to all function prototypes in the header.

Due to the size of `Vec3.cpp`, it was decided not to move the implementations into the header but to rename the file `Vec3.inl` and to include it at the bottom of the header.

**`Vec3.h`**
```cpp
#ifndef VEC3_H__
#define VEC3_H__
/**
 * Three dimensional, double type vector
 */
struct Vec3 {
    // Storage
    double x,y,z;
    // Constructor
    Vec3(const double &a = 0, const double &b = 0, const double &c = 0){
        x=a;
        y=b;
        z=c;
    }
    // Arithmetic operator prototypes
    inline Vec3 operator+(const Vec3 &vec) const;
    inline Vec3& operator+=(const Vec3 &vec);
    // etc...
};
#include "Vec3.inl"
#endif // VEC3_H__
```

**`Vec3.inl`**
```cpp
// Arithmetic operator implementation
inline Vec3 Vec3::operator+(const Vec3 &vec) const {
    return Vec3(
        x + vec.x,
        y + vec.y,
        z + vec.z);
}

inline tVect& tVect::operator+=(const Vec3 &vec) {
    x += vec.x;
    y += vec.y;
    z += vec.z;
    return *this;
}
// etc...
```

After applying this optimisation to the project's vector and quaternion classes, they no longer appeared within the profile instead the self cost of calling functions increased slightly.

When benchmarking [the project before and after this optimisation](https://github.com/gnomeCreative/HYBIRD/commit/8cf8b84ab89d18b10ddce908d4655db409794f19), a 1.87x speedup had been achieved. 


## The Technical Detail

When a compiler inlines a function call, it removes the instruction to call the function (typically `call`) and replaces it with the full implementation of the function. This removes the cost of calling the function, with the trade-off of increasing the compiled binary's size (the function's implementation is duplicated everywhere it has been inlined).

