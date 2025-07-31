---
layout: post
title: Case Study: HYBIRD
date: 2025-07-11 09:00:00
author: Robert Chisholm (Chair)
categories: case-study
---

[HYBIRD](https://github.com/gnomeCreative/HYBIRD) is a simulation framework for modelling interactions between solid particles and fluids implemented with C++ and OpenMP. It couples both a Discrete Element Method (DEM) and a Lattice Boltzmann Method (LBM) solver, filling a niche between widely used scientific packages that exclusively process either DEM or LBM.

<!-- Image/Video? -->

In a pattern common to much research software, HYBIRD was developed as part of a PhD project by a student without formal programming training, however has since been adopted by students and post-docs far beyond the initial scope.

HYBIRD successfully applied to the University of Sheffield's (internal) RSE Call for Proposals 2024, and I was assigned to review the software's performance and investigate a GPU implementation. The early work on this project, which involved reviewing the performance of the existing code, provides a case-study of the impact of applying the SIG-RPC principles to research software.

HYBIRD is provided with several (small) tutorial models, I was asked to use these as representative workloads.

## Inlining

I initially profiled the "Granular Column" tutorial with the function level profiler [GProf](). This tutorial simulates a column of ~N <!-- How many?? --> particles collapsing (pure DEM).

<!-- image/video of start/finish? -->

This produced the below results

<!-- todo re-run to get fresh results-->
```GProf
  %   cumulative   self              self     total           
 time   seconds   seconds    calls   s/call   s/call  name    
 13.75     16.38    16.38 125676749836     0.00     0.00  tVect::operator+(tVect const&) const
 12.60     31.40    15.02 94039864750     0.00     0.00  tVect::operator*(double const&) const
 10.02     43.34    11.94 65081331     0.00     0.00  elmt::correct(double const*, double const*)
  8.54     53.52    10.18 4068178008     0.00     0.00  DEM::particleParticleCollision(particle const*....
  7.62     62.60     9.08 2392019595     0.00     0.00  elmt::predict(double const*, double const*)
  5.66     69.34     6.74 41323211762     0.00     0.00  tVect::operator-(tVect const&) const
  5.45     75.84     6.50 36306445222     0.00     0.00  quaternion::operator*(double const&) const
  4.74     81.49     5.65 7264658795     0.00     0.00  project(tVect, quaternion)
  4.60     86.97     5.48 32022205839     0.00     0.00  quaternion::operator+(quaternion const&) const
  3.61     91.27     4.30  2380119     0.00     0.00  DEM::evaluateForces()
  2.25     93.95     2.68 16726846623     0.00     0.00  tVect::cross(tVect const&) const
...
```

Straight away from this profile, it's clear that there are classes `tVect` and `quaternion`, with mathematical operations that are being called billions of times.

Looking at the implementation of the most expensive, [`tVect::operator+()`](https://github.com/gnomeCreative/HYBIRD/blob/a0cd4dd3c090af318a2c205e4a3856126fb972c9/src/myvector.cpp), it's clearly a tiny function that should be inexpensive.

```c++
tVect tVect::operator+(const tVect& vec) const {
    return tVect(
        x+vec.x,
        y+vec.y,
        z+vec.z);
}
```

As the function has been defined in `myvector.cpp`, it's implementation won't be visible when it's called from other `.cpp` files. This means that it cannot be inlined, and every call to the function will incur a call overhead. Due to the function being so small, and called so frequently, that call overhead is likely to be substantial relative to the cost of the function's computation.

Due to there being so many similar functions in this source file, it was easiest to [inline the whole file](https://github.com/gnomeCreative/HYBIRD/commit/8cf8b84ab89d18b10ddce908d4655db409794f19) by marking each function definition `inline`, renaming the file as `myvector.inl` (so that it is not directly compiled) and including it at the end of `myvector.h`.

When profiled a second time, all of the inlined functions have disappeared. As they are now implemented in-line, they no longer have a function call. If they are still visible, the compiler may have decided to not inline them, this could be because they are too expensive or you've not enabled compiler optimisations.

```GProf
  %   cumulative   self              self     total           
 time   seconds   seconds    calls   s/call   s/call  name    
 57.27     57.75    57.75 204587605     0.00     0.00  elmt::correct(double const*, double const*)
 13.10     70.96    13.21 4068178008     0.00     0.00  DEM::particleParticleCollision(particle const*,...
 10.94     81.99    11.03 2392019595     0.00     0.00  elmt::predict(double const*, double const*)
  8.24     90.30     8.31  2380119     0.00     0.00  DEM::evaluateForces()
  3.85     94.18     3.88 4295241157     0.00     0.00  DEM::FRtangentialContact(tVect const&, double con...
  1.95     96.15     1.97  2380119     0.00     0.00  DEM::particleParticleContacts()
  0.83     96.99     0.84 2392020600     0.00     0.00  particle::updateCorrected(elmt const&, std::vector...
  0.66     97.66     0.67 4295249450     0.00     0.00  DEM::normalContact(double const&, double const&,...
  0.56     98.22     0.56  2380119     0.00     0.00  DEM::wallParticleContacts()
  0.56     98.78     0.56    20001     0.00     0.00  DEM::discreteElementStep()
  0.48     99.26     0.48 227071442     0.00     0.00  DEM::wallParticleCollision(wall*, particle const*,...
...
```

Overall this quick change reduced the runtime of the small tutorial from 1422 to 760 seconds, a 1.87x speedup!

*An alternate option would have been to replace the home-made vector classes with an external library such as [GLM](https://github.com/g-truc/glm), which has been implemented by experienced performance-aware programmers.*

## Redundant Code

## Small IO Operations

## OpenMP

Alongside this

## Conclusion

Robert Chisholm

SIG-RPC Chair

[sig-rpc-managers@society-rse.org](mailto:sig-rpc-managers@society-rse.org)
