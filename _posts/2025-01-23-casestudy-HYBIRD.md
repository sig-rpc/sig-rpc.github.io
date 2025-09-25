---
layout: post
title: Case Study: HYBIRD
date: 2025-07-11 09:00:00
author: Robert Chisholm (Chair)
categories: case-study
---

[HYBIRD](https://github.com/gnomeCreative/HYBIRD) is a simulation framework for modelling interactions between solid particles and fluids implemented with C++ and OpenMP. It couples both a Discrete Element Method (DEM) and a Lattice Boltzmann Method (LBM) solver, filling a niche between widely used scientific packages that exclusively process either DEM or LBM.

<!-- Image/Video? -->

In a pattern common to much research software, HYBIRD was developed as part of a PhD project by a student without formal programming training, however has since been adopted by students and post-docs and extended far beyond the initial scope.

HYBIRD successfully applied to the University of Sheffield's (internal) RSE Call for Proposals 2024, and I was assigned to review the software's performance and investigate a GPU implementation. The early work on this project, which involved reviewing the performance of the existing code, provides a case-study of the impact of applying the SIG-RPC principles to research software.

HYBIRD is provided with several (small) tutorial models, I was asked to use these as representative workloads.

## Inlining

I initially profiled the granular column tutorial with the function level profiler [GProf](/profiler/cpp/gprof/) (and OpenMP disabled). This tutorial simulates a column of 1005 particles collapsing (pure DEM).

<!-- image/video of start/finish? -->

The start of GProf's output is shown below, it shows the 20 most expensive functions:

```GProf
Each sample counts as 0.01 seconds.
  %   cumulative   self                self     total           
 time   seconds   seconds    calls     s/call   s/call  name    
  9.60    167.59   167.59 4068178008     0.00     0.00  DEM::particleParticleCollision(particle const*, particle const*, tVect const&, Elongation*)
  8.66    318.68   151.08 153054682294   0.00     0.00  tVect::operator+(tVect const&) const
  6.38    429.95   111.27 12659255385    0.00     0.00  tVect::norm() const
  5.92    533.25   103.30 2392019595     0.00     0.00  elmt::predict(double const*, double const*)
  5.88    635.80   102.55 117157305920   0.00     0.00  tVect::operator*(double const&) const
  5.58    733.22    97.42 9568078380     0.00     0.00  project(tVect, quaternion)
  5.30    825.80    92.58 2392019595     0.00     0.00  elmt::correct(double const*, double const*)
  4.78    909.21    83.40 47840391900    0.00     0.00  quaternion::operator+(quaternion const&) const
  4.00    979.01    69.81  2380119       0.00     0.00  DEM::particleParticleContacts()
  3.61   1041.96    62.95 23934514861    0.00     0.00  tVect::reset()
  3.55   1103.95    61.99 45963670642    0.00     0.00  tVect::operator-(tVect const&) const
  3.40   1163.29    59.33 47840391900    0.00     0.00  quaternion::operator*(double const&) const
  3.32   1221.25    57.97  2380119       0.00     0.00  DEM::evaluateForces()
  2.99   1273.52    52.27 4784039190     0.00     0.00  quaternion::normalize()
  2.99   1325.73    52.21 16726846623    0.00     0.00  tVect::cross(tVect const&) const
  2.61   1371.27    45.54 4295249450     0.00     0.00  DEM::normalContact(double const&, double const&, double const&, double const&) const
  2.60   1416.69    45.41                               tVect::operator-=(tVect const&)
  2.39   1458.37    41.68 16727253648    0.00     0.00  tVect::operator+=(tVect const&)
  2.13   1495.58    37.21 4295241157     0.00     0.00  DEM::FRtangentialContact(tVect const&, double const&, double const&, double const&, double const&, Elongation*, double const&, double const&, double const&)
  1.28   1517.96    22.38 6460471666     0.00     0.00  tVect::operator/(double const&) const
  ...
```

The first thing which stands out are 13 of the 20 functions listed are members of `tVect` and `quaternion`, mathematical operations that are being called billions of times. Combined they occupy nearly 50% of the runtime. It's unusual for these kinds of functions to be listed, with their calls counted, within an optimised build's profile.

Looking at the implementation of the most expensive of these, [`tVect::operator+()`](https://github.com/gnomeCreative/HYBIRD/blob/a0cd4dd3c090af318a2c205e4a3856126fb972c9/src/myvector.cpp), it's clearly a tiny function that should be inexpensive.

```c++
tVect tVect::operator+(const tVect& vec) const {
    return tVect(
        x+vec.x,
        y+vec.y,
        z+vec.z);
}
```

As the function has been defined in `myvector.cpp`, it's implementation won't be visible when it's called from other `.cpp` files in the codebase. This means that most of the time it cannot be inlined, and every call to the function will incur a call overhead. Due to the function being so small, that call overhead is likely to be substantial relative to the cost of the function's computation. With them each being called billions of times, that's potentially alot of call overhead.

With there being so many similar functions in this source file, it was easiest to [inline](/optimisation/cpp/inlining/) the whole file by marking each function definition `inline`, renaming the file to `myvector.inl` (so that it is not directly compiled) and including it at the end of `myvector.h` ([changes shown here](https://github.com/gnomeCreative/HYBIRD/commit/8cf8b84ab89d18b10ddce908d4655db409794f19)).

When profiled a second time, all of the inlined functions have disappeared. As they are now implemented in-line, they no longer have a function call. If they are still visible, the compiler may have decided to not inline them (`inline` is only a compiler hint), this could be because they are too expensive or you've not enabled compiler optimisations.

```GProf
Flat profile:

Each sample counts as 0.01 seconds.
  %   cumulative   self                self     total           
 time   seconds   seconds    calls     s/call   s/call  name    
 27.76    228.48   228.48 4068178008     0.00     0.00  DEM::particleParticleCollision(particle const*, particle const*, tVect const&, Elongation*)
 18.25    378.71   150.23 2392019595     0.00     0.00  elmt::correct(double const*, double const*)
 15.82    508.95   130.24 2392019595     0.00     0.00  elmt::predict(double const*, double const*)
 14.60    629.11   120.15  2380119       0.00     0.00  DEM::evaluateForces()
 10.77    717.79    88.68 4295241157     0.00     0.00  DEM::FRtangentialContact(tVect const&, double const&, double const&, double const&, double const&, Elongation*, double const&, double const&, double const&)
  2.35    737.10    19.31  2380119       0.00     0.00  DEM::particleParticleContacts()
  2.13    754.65    17.55  2380119       0.00     0.00  DEM::wallParticleContacts()
  1.66    768.31    13.66 4295249450     0.00     0.00  DEM::normalContact(double const&, double const&, double const&, double const&) const
  1.35    779.42    11.11 2392019595     0.00     0.00  particle::updatePredicted(elmt const&, std::vector<std::vector<tVect, std::allocator<tVect> >, std::allocator<std::vector<tVect, std::allocator<tVect> > > > const&)
  1.20    789.27     9.85 2392020600     0.00     0.00  particle::updateCorrected(elmt const&, std::vector<std::vector<tVect, std::allocator<tVect> >, std::allocator<std::vector<tVect, std::allocator<tVect> > > > const&)
  0.96    797.15     7.89 227071442      0.00     0.00  DEM::wallParticleCollision(wall*, particle const*, double const&, Elongation*)
  0.94    804.92     7.77 790777998      0.00     0.00  wall::dist(tVect) const
  0.89    812.27     7.35    20001       0.00     0.04  DEM::discreteElementStep()
  0.42    815.75     3.48  2380119       0.00     0.00  DEM::updateParticlesCorrected()
  0.20    817.42     1.67  2380119       0.00     0.00  DEM::predictor()
  0.18    818.90     1.48                               node::scatterMass(double&)
  0.11    819.78     0.88  2380119       0.00     0.00  DEM::corrector()
  0.10    820.63     0.85                               DEM::objectParticleCollision(object*, particle const*, tVect const&, Elongation*)
  0.10    821.44     0.81                               elmt::translate(tVect const&)
  0.09    822.19     0.75                               elmt::resetVelocity()
...
```

**Overall this quick change, inlining all the small mathematical methods, reduced the runtime of the granular column tutorial from 28m5s to 14m24s. A 1.95x speedup!**

*An alternate option would have been to replace the home-made vector classes with an external library such as [GLM](https://github.com/g-truc/glm), which has been implemented by experienced performance-aware programmers.*

## Redundant Code

## Small IO Operations

## OpenMP

Alongside this

## Conclusion

Robert Chisholm

SIG-RPC Chair

[sig-rpc-managers@society-rse.org](mailto:sig-rpc-managers@society-rse.org)
