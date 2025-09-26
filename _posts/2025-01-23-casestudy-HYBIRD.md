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

HYBIRD successfully applied to the University of Sheffield's (internal) [RSE Call for Proposals](https://rse.shef.ac.uk/collaboration/RSEtime/2024/) in 2024, and I was assigned to review the software's performance and investigate a GPU implementation. The early work on this project, which involved reviewing the performance of the existing code, provides a case-study of the impact of applying the SIG-RPC principles to research software.

HYBIRD is provided with several (small) tutorial models, I was asked to use these as representative workloads.

## Inlining

I initially profiled their [granular column tutorial](https://github.com/gnomeCreative/HYBIRD/wiki/Tutorial-4) with the function level profiler [GProf](/profiler/cpp/gprof/) (and OpenMP disabled). This tutorial simulates a column of 1005 particles collapsing (pure DEM).

<!-- image/video of start/finish? -->

The start of GProf's output is shown below, it shows the 20 most expensive functions:

<!-- gprof with commit a0cd4dd and OpenMP disabled (via modifying CMake source) -->
```GProf
Flat profile:

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

<!-- gprof with commit 8cf8b84 and OpenMP disabled (via modifying CMake source) -->
<!-- not sure if this is correct, although it looks so, the job finished after i'd downloaded this -->
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

**Overall this quick change, inlining all the small mathematical methods, reduced the runtime of the granular column tutorial from 28m5s to 14m24s. A 1.95x speedup to the parallel code!**

*An alternate option would have been to replace the home-made vector classes with an external library such as [GLM](https://github.com/g-truc/glm), which has been implemented by experienced performance-aware programmers.*

## Redundant Code

With no obvious problems remaining in the previous tutorial's profile, it was next suggested I profile their [granular flow tutorial](https://github.com/gnomeCreative/HYBIRD/wiki/Tutorial-3). This tutorial simulates a Newtonian fluid of 50 small particles on an incline (pure LBM).

So I repeated the earlier process, profiling with [GProf](/profiler/cpp/gprof/). Although this time I failed to disable OpenMP.

<!-- gprof with commit 319dff5 and OpenMP enabled -->
```GProf
Each sample counts as 0.01 seconds.
  %   cumulative   self              self     total           
 time   seconds   seconds    calls   s/call   s/call  name    
 28.73     16.42    16.42  5451435     0.00     0.00  node::computeApparentViscosity(double const*, FluidMaterial const&)
 23.36     29.77    13.35 16000012     0.00     0.00  void std::__heap_select<std::reverse_iterator<__gnu_cxx::__normal_iterator<double*, std::vector<double, std::allocator<double> > > >, __gnu_cxx::__ops::_Iter_less_iter>(std::reverse_iterator<__gnu_cxx::__normal_iterator<double*, std::vector<double, std::allocator<double> > > >, std::reverse_iterator<__gnu_cxx::__normal_iterator<double*, std::vector<double, std::allocator<double> > > >, std::reverse_iterator<__gnu_cxx::__normal_iterator<double*, std::vector<double, std::allocator<double> > > >, __gnu_cxx::__ops::_Iter_less_iter)
 14.58     38.10     8.33  5372781     0.00     0.00  node::computeEquilibrium(double*)
 11.08     44.43     6.33      156     0.04     0.04  LB::getZ(unsigned int const&) const
 10.52     50.44     6.01  7072394     0.00     0.00  node::addForce(double*, tVect const&)
  5.53     53.60     3.16  7512840     0.00     0.00  node::reconstruct()
  1.92     54.70     1.10  6324299     0.00     0.00  node::solveCollision(double const*)
  0.98     55.26     0.56  5242043     0.00     0.00  node::shiftVelocity(tVect const&)
  0.98     55.82     0.56  2000001     0.00     0.00  LB::streaming(std::vector<wall, std::allocator<wall> >&, std::vector<object, std::allocator<object> >&)
  0.89     56.33     0.51  9011735     0.00     0.00  node::store()
  0.31     56.51     0.18  4000003     0.00     0.00  LB::cleanLists()
  0.19     56.62     0.11       68     0.00     0.00  node::initialize(double const&, tVect const&, double const&, double const&, tVect const&, double const&, tVect const&)
  0.19     56.73     0.11       68     0.00     0.00  node::setEquilibrium(tVect const&, double const&, tVect const&)
  0.17     56.83     0.10                             node::addForceTRT(tVect const&)
  0.16     56.92     0.09                             node::computeEquilibriumTRT(double*, double*)
  0.07     56.96     0.04  2000001     0.00     0.00  LB::updateInterface()
  0.07     57.00     0.04  2000001     0.00     0.00  LB::latticeBoltzmannFreeSurfaceStep()
  0.07     57.04     0.04                             node::solveCollisionTRT(double const*, double const*, double const&)
  0.03     57.06     0.02 19856644     0.00     0.00  node::isFluid() const
  0.03     57.08     0.02  2000001     0.00     0.00  LB::updateMutants(double&)
...
```

The second most expensive method here, is `std::__heap_select` an internal C++ method, to investigate that further, I looked for it later in the GProf analysis. Beyond the flat profile, there is the call graph, which described the call tree of the program, sorted by the total amount of time spent in each function.

Below is the first section of the call-graph, which contains `std::_heap_select`.

<!-- gprof with commit 319dff5 and OpenMP enabled -->
```GProf
...
-----------------------------------------------
                             16000012             void std::__heap_select<std::reverse_iterator<__gnu_cxx::__normal_iterator<double*, std::vector<double, std::allocator<double> > > >, __gnu_cxx::__ops::_Iter_less_iter>(std::reverse_iterator<__gnu_cxx::__normal_iterator<double*, std::vector<double, std::allocator<double> > > >, std::reverse_iterator<__gnu_cxx::__normal_iterator<double*, std::vector<double, std::allocator<double> > > >, std::reverse_iterator<__gnu_cxx::__normal_iterator<double*, std::vector<double, std::allocator<double> > > >, __gnu_cxx::__ops::_Iter_less_iter) [3]
                0.00    0.00       1/16000012     LB::latticeBolzmannInit(std::vector<cylinder, std::allocator<cylinder> >&, std::vector<wall, std::allocator<wall> >&, std::vector<particle, std::allocator<particle> >&, std::vector<object, std::allocator<object> >&, bool, bool) [10]
                1.67    4.51 2000001/16000012     LB::latticeBolzmannStep(std::vector<elmt, std::allocator<elmt> >&, std::vector<particle, std::allocator<particle> >&, std::vector<wall, std::allocator<wall> >&, std::vector<object, std::allocator<object> >&) [5]
                1.67    4.51 2000001/16000012     LB::latticeBoltzmannFreeSurfaceStep() [6]
               10.01   27.08 12000009/16000012     LB::cleanLists() [4]
[3]     86.5   13.35   36.10 16000012+16000012 void std::__heap_select<std::reverse_iterator<__gnu_cxx::__normal_iterator<double*, std::vector<double, std::allocator<double> > > >, __gnu_cxx::__ops::_Iter_less_iter>(std::reverse_iterator<__gnu_cxx::__normal_iterator<double*, std::vector<double, std::allocator<double> > > >, std::reverse_iterator<__gnu_cxx::__normal_iterator<double*, std::vector<double, std::allocator<double> > > >, std::reverse_iterator<__gnu_cxx::__normal_iterator<double*, std::vector<double, std::allocator<double> > > >, __gnu_cxx::__ops::_Iter_less_iter) [3]
               16.42    0.00 5451435/5451435     node::computeApparentViscosity(double const*, FluidMaterial const&) [8]
                8.33    0.00 5372781/5372781     node::computeEquilibrium(double*) [9]
                6.01    0.00 7072394/7072394     node::addForce(double*, tVect const&) [13]
                3.16    0.00 7512840/7512840     node::reconstruct() [14]
                1.10    0.00 6324299/6324299     node::solveCollision(double const*) [17]
                0.56    0.00 5242043/5242043     node::shiftVelocity(tVect const&) [18]
                0.51    0.00 9011735/9011735     node::store() [20]
                0.01    0.00 9854995/19856644     node::isFluid() const [30]
                0.00    0.00 22744828/22744828     node::massStream(unsigned int const&) const [45]
                0.00    0.00 22152591/32108542     node::isInterface() const [44]
                             16000012             void std::__heap_select<std::reverse_iterator<__gnu_cxx::__normal_iterator<double*, std::vector<double, std::allocator<double> > > >, __gnu_cxx::__ops::_Iter_less_iter>(std::reverse_iterator<__gnu_cxx::__normal_iterator<double*, std::vector<double, std::allocator<double> > > >, std::reverse_iterator<__gnu_cxx::__normal_iterator<double*, std::vector<double, std::allocator<double> > > >, std::reverse_iterator<__gnu_cxx::__normal_iterator<double*, std::vector<double, std::allocator<double> > > >, __gnu_cxx::__ops::_Iter_less_iter) [3]
-----------------------------------------------
...
```

The call graph is much harder to interpret, but here we can see the focused call to `std::_heap_select` in the middle beginning with `[3]`, the line above it shows the function `LB::cleanLists()` which must be where this call is occurring. This function was shown in the above flat profile, so it's not clear why `std::_heap_select`s cost wasn't included in it's cost.

Regardless, it's worth taking a look at `LB::cleanLists()`:

```cpp
void LB::cleanLists() {
    // sorts the active-node list and removes duplicates
    std::sort(fluidNodes.begin(), fluidNodes.end());
    std::sort(interfaceNodes.begin(), interfaceNodes.end());

    for (int ind = fluidNodes.size() - 1; ind > 0; --ind) {
        node* i = fluidNodes[ind];
        node* j = fluidNodes[ind - 1];
        if (i == j) {
            cout << "duplicate-fluid!" << endl;
            fluidNodes.erase(fluidNodes.begin() + ind);
        }
    }
    for (int ind = interfaceNodes.size() - 1; ind > 0; --ind) {
        node* i = interfaceNodes[ind];
        node* j = interfaceNodes[ind - 1];
        if (i == j) {
            cout << "duplicate-interface!" << endl;
            interfaceNodes.erase(interfaceNodes.begin() + ind);
        }
    }

    // list with active nodes i.e. nodes where collision and streaming are solved
    // solid nodes, particle nodes and gas nodes are excluded
    activeNodes.clear();
    activeNodes.reserve(fluidNodes.size() + interfaceNodes.size());
    activeNodes.insert(activeNodes.end(), fluidNodes.begin(), fluidNodes.end());
    activeNodes.insert(activeNodes.end(), interfaceNodes.begin(), interfaceNodes.end());

}
```

The LBM model within HYBRID stores nodes sparsely, so it has lists `fluidNodes` (nodes filled with fluid), `interfaceNodes` (nodes which form the boundary between fluid and gas) and `activeNodes` a list which contains elements from both the other lists. Gas nodes are not explicitly represented, and these lists hold pointers.

Therefore, it can be understood that `LB::cleanLists()` first sorts the fluid and interface lists, then iterates each of them to remove consecutive duplicate elements before finally combining them to update `activeNodes`. The sorting of the lists here, is probably what's triggering `std::_heap_select`.

This approach to managing duplicates isn't the most naïve (it's avoiding nested iteration), but it would be faster to use a set data-structure ([python set guide](/optimisation/python/set/)<!--@todo-->, until we have a C++ guide). However when informed of this issue the main author of HYBIRD clarified that this validation was redundant, left in from debugging an old issue. As such, I added some pre-processor macros to ensure it was only executed during debug builds ([changes shown here](https://github.com/gnomeCreative/HYBIRD/commit/50d84400a12a937a5697b4d4dc2d54a456d1ef55)).

**This small change, removing redundant validation, reduced the runtime of the granular flow tutorial from 2m4s to 1m9s. A 1.80x speedup to the parallel code! Furthermore, the nature of the removed code would likely scale poorly as the number of particles increased, suggesting it may have enabled higher speedups.**

*It's worth noting that this issue was only prominent in a parallel profile, as it was one of the few methods which did not utilise parallelisation, although it may have become prominent at larger scales.*

## Set

It was at this point, I asked if there were larger problems I could profile (or how I could scale these tutorials up). As the scale of a problem increases, if a bottleneck is reached it will get slower at a faster pace than the surrounding code. This can make it much easier to spot the bottlenecks within a profile.

I was provided with a bespoke example informally called granular collapse continuum. While conceptually similar to the previous tutorial, this scenario began with 2.2 million `activeNodes` (fluid and interface nodes, mostly fluid). We configured the simulation steps so that roughly half would occur **after** the fluid impacted the wall. This ensured we captured the turbulence generated by the collision, when many nodes change state, while avoiding unnecessary data collection and keeping profiling time manageable.

Again, profiling the parallel code with [GProf](/profiler/cpp/gprof/):

@todo profiling output that shows smoothInterface high

Looking through ?????? `LB::smoothenInterface()`, what first stands out is this comment.

```
// ERROR: TAKE OUT FROM CYCLE, THIS IS KILLING PERFORMANCE
```

So an author of the code, knew it was a problem for performance.

I've briefly summarised the algorithm, which removes neighbours of `emptiedNodes` from `fluidNodes` (as they will become `interfaceNodes`) in pseudo-code below

```py
for emptyNode in emptiedNodes:
  for neighbour in emptyNode.neighbours:
    if neighbour.isValid() and neighbour.isFluid():
        newInterfaceNodes.append(neighbour)
        neighbour.redistributeMass()
        # ERROR: TAKE OUT FROM CYCLE, THIS IS KILLING PERFORMANCE
        for fluidNode in fluidNodes:
          if fluidNode == neighbour:
            fluideNodes.erase(fluidNode)
```

The comment clearly denotes the inner loop. In this particular example we had over 2 million nodes, therefore for every fluid neighbour of an emptied node the entire fluid list would be iterated to remove a neighbour if present. However, it's quite possible multiple fluid nodes would share a neighbour, making some of these searches redundant.

To optimise this, we can replace that inner loop, instead inserting `fluidNode` into a set of nodes to be deleted. After the main outer loop completes, we have a set of unique nodes to be deleted, so we can iterate `fluidNodes` once, deleting any nodes that appear in the set. Checking whether a node exists within our set of nodes to be deleted, should almost always be faster than iterating that same data in a list or array ([changes shown here](https://github.com/gnomeCreative/HYBIRD/commit/f86ab23640b59c9befe5e29290359d4e40789245)).

**Through this change using a set data-structure where appropriate, reduced the runtime of the profiled example from 6h51m to 2h23m. A 2.87x speedup to the parallel code!**

## Small IO Operations

## OpenMP

Alongside this profiling and optimisation work, there were several scattered improvements to the OpenMP within the project. These took the form of typical bad habits; overuse of expensive critical sections, and creating many independent parallel sections rather than creating less and sharing them to reduce the creation overhead.

<!--@todo links to relevant OpenMP optimisation pages when they exist -->

## Conclusion

In all there were a large number of optimisations applied to this code-base throughout the project, and two of the optimisations removed algorithms which scaled poorly, so it's difficult to estimate the combined speed-up.

To provide a summary comparison I benchmarked the [tsunami wave tutorial](https://github.com/gnomeCreative/HYBIRD/wiki/Tutorial-5), which is the largest of the provided examples and combines both DEM and LBM, with both the pre- and post-optimisation code. Before optimisation it took ??? to execute to completion, and afterwards it took 4h15m. This equates to a combined ???x speed-up to that particular example. As previously discussed, these speed ups are highly likely to vary across problems due to the nature of the optimisations.

Further to this speed-up, the replacement of ASCII with binary output files would see the previously mentioned ??? improvement to Paraview's loading speed. Additionally improving research productivity when processing the results.

As this work occurred within a funded project, after these optimisations were exhausted the remaining project time was spent re-implementing the code to execute on GPU with CUDA. A functional proof of concept, which was not further profiled or optimised due to time constraints, was completed and validated. As the code heavily relies on double-precision floating point computation, it was found to execute approximately 4x faster than the optimised parallel CPU code on a 3060RTX GPU and approximately 30x faster than CPU on a H100 GPU.

Overall these changes have enabled increased researcher productivity, and larger more complex physical experiments to be simulated computationally. A short quote from the project's PI and HYBIRD's main author [Dr Alessandro Leonoardi](https://sheffield.ac.uk/mac/people/civil-academic-staff/alessandro-leonardi):

> @TODO


Robert Chisholm

SIG-RPC Chair

[sig-rpc-managers@society-rse.org](mailto:sig-rpc-managers@society-rse.org)
