---
layout: post
title: "Case Study: FFEA"
date: 2026-01-15 09:00:01
author: Robert Chisholm (Chair)
categories: [case-study]
language: [C/C++]
excerpt_separator: <!--more-->
---

[FFEA](https://ffea.bitbucket.io/) stands for Fluctuating Finite Element Analysis. It is a research software platform used to model biological molecules as they move and flex under thermal effects. The software itself was first released in 2018, building on methods that had been developed and published since 2010.

Since then, FFEA has seen steady adoption in the academic literature, with around 12 published studies using it, including work led by researchers who were not involved in its development. This level of continuity and uptake is relatively successful for specialised scientific software.

<!--more-->

Like much research code, FFEA reflects the environment in which it was created. It is written in C++ and uses OpenMP for parallelism, and over time it has been worked on by roughly 9 contributors. Most of these were PhD students or postdoctoral researchers on fixed-term contracts. They were experts in their scientific domain but largely self-taught programmers, contributing features or fixes needed for their own research before moving on. The result is a codebase that grew organically, with differences in style, structure, and levels of polish across different parts of the software.

In the second half of 2024, I was asked to help modernise and tidy up the codebase to make it easier for future PhD students to work with. I did this alongside a postdoctoral researcher who had deep knowledge of both FFEA and finite element analysis. Later, in August 2025, after that colleague had left, I was asked to take a closer look at FFEA’s performance, drawing on my previous experience with the code and my background in performance analysis. That work led to two straightforward improvements, which are the focus of this case study.

---

*Part of this research software profiling and optimisation case study was presented during a talk at the [DiRAC & HPC-AI Advisory Council's 7th Annual UK Conference](https://dirac.ac.uk/hpc-ai-advisory-council-7th-annual-uk-conference/), and is available to [watch on YouTube [3:45-10:40]](https://youtu.be/s-Ui4bh39Zc?t=225) ([slides](https://dirac.ac.uk/wp-content/uploads/2025/10/3_Sheffield.pdf)).*

FFEA models two main types of objects. One represents roughly spherical structures, known as blobs, and the other represents elongated structures, known as rods, which are built from chains of nodes. These two concepts are largely separate in the code. To understand performance in realistic conditions, I worked with an active FFEA user to profile one representative example of each.

To begin profiling, I deliberately kept things simple. I turned off OpenMP, which is what allows the software to use multiple CPU cores, and opted to use a basic profiling tool called [`gprof`](/profiler/cpp/gprof/) (which is otherwise not suitable for profiling parallel code). Focusing on single-threaded execution usually makes the results easier to interpret, and improvements found this way almost always benefit parallel runs as well.

The first obstacle appeared sooner than expected. FFEA relied on OpenMP not just for parallelism, but also for its internal timing and reporting. This meant the code would not compile cleanly if OpenMP was disabled. Fixing this required replacing the timing logic with `std::chrono`, a modern and standard timing facility provided by C++ itself. This change took a little effort, but once it was done, I was finally able to build and run a clean serial profile and move on to the actual performance investigation.

As a CMake project, compiling for `gprof` requires passing `-pg` to every compilation stage. Likewise, I disabled OpenMP with an FFEA specific option. Otherwise, compilation proceeded as normal.

```sh
cmake
  -DCMAKE_C_FLAGS=-pg
  -DCMAKE_CXX_FLAGS=-pg
  -DCMAKE_EXE_LINKER_FLAGS=-pg
  -DCMAKE_SHARED_LINKER_FLAGS=-pg
  -DUSE_OPENMP=OFF
  ..
cmake --build .
```

FFEA is then executed as normal, and on successful exit it outputs an additional `gmon.out` file.

This can then be passed, with the FFEA executable to `gprof` to generate a plain text profiling report.

```sh
gprof /path/to/ffea gmon.out > profiling_report.txt
```

At the top of the report is gprof's flat profile, which shows the most expensive functions. This is the most important thing for spotting what should be investigated.

## Profiling Blobs

The blob example that I profiled was one of the test cases `cyl_flexrig`, which represents a single large blob with 226 faces. It's a rather expensive test, taking around 10 minutes to execute in serial, but I'm not familiar with configuring FFEA's inputs and it was suggested to me as an example from one of their early publications.

The flat profile returned by `gprof` has been reproduced below. Negligible cost rows and unimportant function arguments have been replaced with `...`.

```text
Flat profile:

Each sample counts as 0.01 seconds.
  %   cumulative   self              self     total           
 time   seconds   seconds    calls   s/call   s/call  name    
 63.28    373.55   373.55   329186     0.00     0.00  SparseMatrixFixedPattern::apply(std::vector<...> const&, ...) const
 17.81    478.68   105.13      501     0.21     1.15  NoMassCGSolver::solve(std::vector<...>&)
  6.14    514.95    36.27   658372     0.00     0.00  vec3_add_to_scaled(std::vector<...>&, std::vector<...>&, double)
  4.68    542.56    27.61   328685     0.00     0.00  NoMassCGSolver::parallel_apply_preconditioner()
  4.07    566.59    24.03   328685     0.00     0.00  vec3_scale_and_add(std::vector<...>&, std::vector<...>&, double)
  1.86    577.55    10.96 483865299     0.00     0.00  sparse_entry_sources::sum_all_sources()
  ...
```

This doesn't look too unreasonable, all of the methods have names which sound important. Looking a bit closer at each of their implementations, [`SparseMatrixFixedPattern::apply(const std::vector<arr3> &,...)`](https://bitbucket.org/FFEA/ffea/src/854b22f23b6e8b25582d487ab61aa63891e97f75/src/SparseMatrixFixedPattern.cpp?at=master#lines-148) appears worth investigating further. It has been reproduced below, with OpenMP code removed.

```c++
/* Applies this matrix to the given vector 'in', writing the result to 'result'. 'in' is made of 'arr3's */
/* Designed for use in NoMassCGSolver */
void SparseMatrixFixedPattern::apply(const std::vector<arr3> &in, std::vector<arr3> &result) const {
    // To get rid of conditionals, define an array 'num_rows' long, and copy into result at end
    vector<scalar> work_in(num_rows);
    vector<scalar> work_result(num_rows);

    for(int i = 0; i < num_rows / 3; ++i) {
        work_in[3 * i] = in[i][0];
        work_in[3 * i + 1] = in[i][1];
        work_in[3 * i + 2] = in[i][2];
    }

    for (int i = 0; i < num_rows; i++) {
        // Zero array first
        work_result[i] = 0.0;

        for(int j = key[i]; j < key[i + 1]; ++j) {
            work_result[i] += entry[j].val * work_in[entry[j].column_index];
        }
    }

    for(int i = 0; i < num_rows / 3; ++i) {
        result[i][0] = work_result[3 * i];
        result[i][1] = work_result[3 * i + 1];
        result[i][2] = work_result[3 * i + 2];
    }
}
```

To summarise the method

- Allocate two arrays `work_in` and `work_result`.
- Copy data from `in` to `work_in`.
- Compute a result in `work_result` using `work_in`
- Copy `work_result` to `result`

This has a high potential to be a redundant vector allocation and memory movement.
With a bit of investigation there's no sign of a race-condition within OpenMP, it merely looks like the original vector of small `double` arrays (`arr3`), is being unpacked into a vector of `double` (`scalar`) to simplify the array accesses.
Using a reinterpret cast of pointers to the arrays, we can directly access them in the same manner, simplifying the code even further.

```c++
void SparseMatrixFixedPattern::apply(const std::vector<arr3> &in, std::vector<arr3> &result) const {
    const scalar* work_in = reinterpret_cast<const scalar*>(in.data());
    scalar* work_result = reinterpret_cast< scalar*>(result.data());

    for (int i = 0; i < num_rows; i++) {
        work_result[i] = 0;
        for(int j = key[i]; j < key[i + 1]; ++j) {
            work_result[i] += entry[j].val * work_in[entry[j].column_index];
        }
    }
}
```

This reduced the runtime of the (serial) test  from 610 to 534 seconds, and the OpenMP parallel version from 202 to 149 seconds. That's an average of 1.2x speedup.

An updated profile from `gprof`, following the optimisation, is provided below.

```text
Flat profile:
Each sample counts as 0.01 seconds.
  %   cumulative   self              self     total           
 time   seconds   seconds    calls   s/call   s/call  name    
 59.28    319.83   319.83   329186     0.00     0.00  SparseMatrixFixedPattern::apply(std::vector<...> const&, ...) const
 12.57    387.63    67.80      501     0.14     1.05  NoMassCGSolver::solve(std::vector<...>&)
 10.20    442.68    55.05   328685     0.00     0.00  NoMassCGSolver::parallel_apply_preconditioner()
  7.42    482.72    40.04   658372     0.00     0.00  vec3_add_to_scaled(std::vector<...>&, std::vector<...>&, double)
  6.21    516.23    33.51   328685     0.00     0.00  vec3_scale_and_add(std::vector<...>&, std::vector<...>&, double)
  1.98    526.89    10.66 483865299     0.00     0.00  sparse_entry_sources::sum_all_sources()
  ...
```

Surprisingly, it's also improved the self seconds of many of the other most expensive methods. Perhaps removal of the redundant allocations and memory movement reduced pressure on a memory bottleneck elsewhere in the code, for example by reducing processor cache evictions.

We already looked at the other methods and they looked reasonable, so there's nothing more to be done here.

## Profiling Rods

The rod example that I profiled contained a bundle of 125 rods, representative of myofilaments, each made up of 10 segments. It's timestep and duration were again adjusted several orders of magnitude so that it executed in 5 minutes, rather than a full week.

{% figure caption:"PyMol visualisation of the myofilaments example" label:myofilaments %}
![A screenshot of Intel VTune's results flame graph for a hotspots profile.](/assets/case-studies/ffea/myofilaments.png)
{% endfigure %}

The flat profile returned by `gprof`, has been reproduced below. Negligible cost rows and unimportant function arguments have been replaced with `...`.

```text
Flat profile:

Each sample counts as 0.01 seconds.
  %   cumulative   self              self     total           
 time   seconds   seconds    calls  ms/call  ms/call  name    
 53.94      2.12     2.12 36497000     0.00     0.00  rod::Rod::Rod(rod::Rod const&)
 19.85      2.90     0.78 72994000     0.00     0.00  std::vector<...>* std::__do_uninit_copy<...>(...)
 11.45      3.35     0.45 36497000     0.00     0.00  rod::Rod::~Rod()
  2.29      3.44     0.09 20715750     0.00     0.00  rod::get_element_midpoint(...)
  1.78      3.51     0.07 18073000     0.00     0.00  rod::Rod::get_p(int, std::array<float, 3ul>&, bool)
...
```

Even if you're unfamiliar with FFEA's source code and only know some C++, the first three rows of the flat profile should jump out as suspicious.

- 54% of the runtime is spent in `rod::Rod::Rod(rod::Rod const&)`
  - That's a copy constructor for the class `Rod`, and it's being called 36 million times.
- 20% of the runtime is spent in `std::vector::_do_uninit_copy()`.
  - That's an unfriendly internal template method, but it would appear to be related to copying instances of `std::vector`.
- 11% of the runtime is spent in `rod::Rod::~Rod()`
  - Paired with the earlier copy constructor, this shows that 36 million `Rod` objects are being destroyed.
  
From the profile, it quickly became clear that between 65 and 85 percent of the total runtime was spent creating and destroying `Rod` objects. In the profiled case there were only 125 rods, each made up of 10 segments, and these numbers remain constant throughout the simulation. Seeing such a large fraction of the runtime spent on repeatedly creating and discarding objects that never change in number was a strong signal that this part of the code was worth investigating further.

The `Rod` class is defined in [`include/rod_structure.h`](https://bitbucket.org/FFEA/ffea/src/854b22f23b6e8b25582d487ab61aa63891e97f75/include/rod_structure.h?at=master), and starting at line 51 we find the class definition.

<details markdown="1">
  <summary>View <code class="language-plaintext highlighter-rouge">Rod</code> member variable declarations.</summary>
```c++
    struct Rod
    {
        /** Rod metadata **/
        int length;                   /** The length, L, of an array in the rod (x3 the number of nodes, N, in x, y and z) */
        int num_nodes;                /** The number of nodes in the rod */
        int num_rods;                 /** When the system is set up in ffeatools, this will be set */
        int rod_no;                   /** Each rod will be created with a unique ID */
        int line_start = 0;           /** Keeps track of whether the header file has been read */
        double rod_version = 999.999; /** If version number unknown, assume latest **/
        bool computed_rest_energy = false;
        int num_vdw_sites = 0;        /** The number of attractive van der Waals sites on the rod. Arranged by site index */

        /** Global simulation parameters - eventually read in from the .ffea file **/
        float viscosity = 0.6913 * pow(10, -3) / (mesoDimensions::pressure * mesoDimensions::time);  // denominator: poiseuille
        float timestep = 1e-12 / mesoDimensions::time;
        float kT = 0;
        float perturbation_amount = 0.001 * pow(10, -9) / mesoDimensions::length; /** Amount by which nodes are perturbed during numerical differentiation. May want to override with a local value depending on the scale of the simulation. **/
        int calc_noise = 0;
        int calc_steric = 0;  // Repulsive overlap of elements
        int calc_vdw = 0;   // Attractive protein-protein interactions
        int pbc = 0;
        float max_steric_energy = 50;    /** Potential energy when two rod elements are fully overlapped [energy units] **/
        std::string flow_profile;       // the type of background flow experienced by the rod (set by .ffea file)
        float flow_velocity[3] = {0};   // background flow imposed on the rod (set by .ffea file)
        float shear_rate = 0;           //

        float translational_friction;
        float rotational_friction; /** these will have to be changed when I end up implementing per-element radius, to be computed on-the-fly instead most likely **/

        /** Each set of rod data is stored in a single, c-style array, most of which go as {x,y,z,x,y,z...} */

        std::vector<float> equil_r;                              /** L, Equilibrium configuration of the rod nodes. */
        std::vector<float> equil_m;                              /** L, Equilibrium configuration of the material frame. */
        std::vector<float> current_r;                            /** L, Current configuration of the rod nodes. */
        std::vector<float> current_m;                            /** L, Current configuration of the material frame. */
        std::vector<float> internal_perturbed_x_energy_positive; /** L, Energies associated with the perturbations we do in order to get dE. Given as [stretch, bend, twist, stretch, bend, twist...]**/
        std::vector<float> internal_perturbed_y_energy_positive; // L
        std::vector<float> internal_perturbed_z_energy_positive; // L
        std::vector<float> internal_twisted_energy_positive;     // L
        std::vector<float> internal_perturbed_x_energy_negative; /** L, Stretch, bend, twist, stretch, bend, twist... */
        std::vector<float> internal_perturbed_y_energy_negative; // L
        std::vector<float> internal_perturbed_z_energy_negative; // L
        std::vector<float> internal_twisted_energy_negative;     // L
        std::vector<float> material_params;                  /** L, Stretch, twist, radius, stretch, twist, radius... **/
        std::vector<float> B_matrix;                         /** L+L/3, Contents of the bending modulus matrix for each node, as a 1-d array. Given as [a_1_1, a_1_2, a_2,1, a_2_2, a_1_1...]. **/
        std::vector<float> steric_perturbed_energy_positive; // Length 2L array: energies from steric interactions at the start (0) and end (1) nodes of each rod element, i. Given as [xi0 yi0 zi0, xi1 yi1 zi1, ...]
        std::vector<float> steric_perturbed_energy_negative; // 2L
        std::vector<float> steric_energy;                    // L, steric repulsion energy of elements (should be L/3 array, but previous decisions make it easier this way; xi=yi=zi, and so on)
        std::vector<float> steric_force;                     // L, steric repulsive force interpolated onto nodes from elements [x, y, z, ...]
        std::vector<int> num_steric_nbrs; // L/3. Keeps track of how many neighbours each rod element has.
        std::vector<float> vdw_energy; // L
        std::vector<float> vdw_force; // L
        std::vector<int> num_vdw_nbrs;
        std::vector<float> vdw_site_pos;     // Length = num_vdw_sites * 3. Must be a vector since it has to allow for there being no VDW sites on the rod.
        std::vector<float> applied_forces;              /** L, Another [x,y,z,x,y,z...] array, this one containing the force vectors acting on each node in the rod. **/
        std::vector<bool> pinned_nodes;                 /** L/3, This array is the length of the number of nodes in the rod, and it contains a boolean stating whether that node is pinned (true) or not (false). **/

        bool interface_at_start = false;    /** if this is true, the positioning of the start node is being handled by a rod-blob interface, so its energies will be ignored. **/
        bool interface_at_end = false;      /** if this is true, the positioning of the end node is being handled by a rod-blob interface, so its energies will be ignored. **/
        bool restarting = false;            /** If this is true, the rod will skip writing a frame of the trajectory (this is normally done so that the trajectory starts with correct box positioning) **/

        std::vector<std::vector<InteractionData>> steric_nbrs; /** Steric interaction neighbour list **/
        std::vector<std::vector<InteractionData>> vdw_nbrs;
        std::vector<VDWSite> vdw_sites;  // Attractive van der Waals binding sites that lie along the rod

        /** Unit conversion factors - the input\output files are in SI, but internally it uses FFEA's units as determined in dimensions.h **/
        float bending_response_factor;
        float spring_constant_factor;
        float twist_constant_factor;
        float viscosity_constant_factor;

        std::string rod_filename;
        std::string vdw_filename;
        FILE *file_ptr;
        int frame_no = 0;
        int step_no = 0;  // ! - redundant?
        // ... Member functions removed from this snippet
    }
```
</details>

Looking at the class's member variables there are around 30 scalars, 28 `std::vector` and several `std::string`. This is not a trivial structure, like a small matrix or vector, it will be expensive to make copies. And those 28 vectors, probably explain the earlier `std::vector::_do_uninit_copy()` given we already know instances of `Rod` are being copied.

Looking more closely at the `Rod` class, it became clear that it is a fairly heavyweight object. It contains around 30 scalar values, 28 `std::vector` members, and several `std::string`. This is not a simple data structure like a small matrix or a coordinate vector, and copying it is inherently expensive.

Those 28 vectors are especially telling, they neatly explain the earlier appearance of `std::vector::_do_uninit_copy()` in the profile, given that we know millions of `Rod` objects are being copied.

At the end of the `Rod`, we find it's function declarations.

<details markdown="1">
  <summary>View <code class="language-plaintext highlighter-rouge">Rod</code> member variable declarations.</summary>
```c++
    struct Rod
    {
        // ... Member variables removed from this snippet
        Rod(int length, int set_rod_no);
        Rod(std::string path, int set_rod_no);
        Rod set_units();
        Rod compute_rest_energy();
        Rod do_timestep(std::shared_ptr<std::vector<RngStream>> &rng);
        Rod add_force(const float4 &force, int node_index);
        Rod pin_node(bool pin_state, int node_index);
        Rod load_header(std::string filename);
        Rod load_contents(std::string filename);
        Rod load_vdw(const std::string filename);
        Rod write_frame_to_file();
        Rod write_mat_params_vector(const std::vector<float> &vec, float stretch_scale_factor, float twist_scale_factor, float length_scale_factor);
        Rod change_filename(std::string new_filename);
        Rod equilibrate_rod(std::shared_ptr<std::vector<RngStream>> &rng);
        Rod translate_rod(std::vector<float> &r, const float3 &translation_vec);
        Rod rotate_rod(const float3 &euler_angles);
        Rod scale_rod(float scale);
        Rod get_centroid(const std::vector<float> &r, float3 &centroid);
        Rod get_min_max(const std::vector<float> &r, OUT float3 &min, float3 &max);
        Rod get_p(int index, OUT float3 &p, bool equil);
        Rod get_r(int node_index, OUT float3 &r, bool equil);
        float get_radius(int node_index);
        float contour_length();
        float end_to_end_length();
        int get_num_nbrs(int element_index, const std::vector<std::vector<InteractionData>> &nbr_list);
        int get_num_vdw_sites();
        int get_num_nodes();
        Rod check_nbr_list_dim(std::vector<std::vector<InteractionData>> &nbr_list);
        void reset_nbr_list(std::vector<std::vector<InteractionData>> &nbr_list);
        Rod print_node_positions();
        std::vector<float> net_steric_force_nbrs(int elem_id);
        std::vector<float> net_vdw_force_nbrs(int elem_id);
        void do_steric();
        void do_vdw();
    }
```
</details>

Quite quickly it became visible that the majority of the member functions of `Rod` return a `Rod` object, which probably explains all the copies.

The implementation of those member functions can be found in [`src/rod_structure.cpp`](https://bitbucket.org/FFEA/ffea/src/854b22f23b6e8b25582d487ab61aa63891e97f75/src/rod_structure.cpp?at=master), where inside `Rod::set_units()` we find a comment that explains the reason for returning a `Rod`.

```cpp
    Rod Rod::set_units()
    {
        // ... Other code removed for conciseness
        
        return *this; /** Return a pointer to the object itself instead of void. Allows for method chaining! **/
    }
```

Except `*this` de-references the pointer, instead returning a copy of the object rather than a pointer to it. Due to how the `Rod` class is structured method chaining is not possible, as each instance has it's own copy of the data. By checking every call of these functions, it was possible to confirm that method chaining was not being attempted. Hence, each of these returns could be replaced with `void`.

On repeating the profile with this fix applied, `gprof` now shows a wider range of functions primarily handling maths as the most expensive components which appears more reasonable. This has been reproduced below, with unimportant rows and function arguments replace by `...`.

```text
Flat profile:

Each sample counts as 0.01 seconds.
  %   cumulative   self              self     total           
 time   seconds   seconds    calls  ms/call  ms/call  name    
 17.65      0.21     0.21 28542690     0.00     0.00  rod::absolute(std::array<float, 3ul> const&)
 12.61      0.36     0.15    85250     0.00     0.00  World::update_rod_vdw_nbr_lists(rod::Rod*, rod::Rod*, SSINT_matrix*)
 10.92      0.49     0.13  6905250     0.00     0.00  rod::set_steric_nbrs(...)
  7.56      0.58     0.09  6107299     0.00     0.00  rod::normalize(std::array<float, 3ul> const&, std::array<float, 3ul>&)
  5.04      0.64     0.06 20715750     0.00     0.00  rod::get_element_midpoint(...)
  4.20      0.69     0.05  1262023     0.00     0.00  boost::detail::function::function_obj_invoker<...>::invoke(...)
  4.20      0.74     0.05   392667     0.00     0.00  FFEA_input_reader::parse_tag(...)
  3.36      0.78     0.04 18073000     0.00     0.00  rod::Rod::get_r(int, std::array<float, 3ul>&, bool)
  2.52      0.81     0.03  5116606     0.00     0.00  std::vector<...>(...)
  2.52      0.84     0.03  4361093     0.00     0.00  rod::get_p_i(...)
  2.52      0.87     0.03  4331875     0.00     0.00  rod::VDWSite::update_position(...)
  2.52      0.90     0.03  1876875     0.00     0.00  rod::get_rotation_matrix(...)
  2.52      0.93     0.03   409146     0.00     0.00  std::vector<...>::vector<...>()
  2.52      0.96     0.03   198000     0.00     0.00  rod::get_stretch_energy(float, std::array<float, 3ul>&, std::array<float, 3ul>&)
...
```

There's no further obvious issues here, although `FFEA_input_reader` occupying 4% of the runtime could be concerning if we weren't profiling a small test case.

Checking the new most expensive function`rod::absolute(std::array<float, 3ul> const&)` (within [`src/rod_math_v9.cpp`](https://bitbucket.org/FFEA/ffea/src/854b22f23b6e8b25582d487ab61aa63891e97f75/src/rod_math_v9.cpp?at=master#lines-365)) we find a typical calculation of vector magnitude, albeit with a call to an inlined method which checks for `nan` and `inf`. This could be pushed behind a debug or validation macro if chasing every ounce of performance, however it's likely a negligible impact to performance in the grand scheme.

```cpp
    /**
 Get the absolute value of a vector.
*/
    float absolute(const float3 &in)
    {
        float absolute = sqrt(in[x] * in[x] + in[y] * in[y] + in[z] * in[z]);
        not_simulation_destroying(absolute, "Absolute value is simulation destroying.");
        return absolute;
    }
```

The second most expensive function `World::update_rod_vdw_nbr_lists(rod::Rod*, rod::Rod*, SSINT_matrix*)` (within [`src/World.cpp`](https://bitbucket.org/FFEA/ffea/src/854b22f23b6e8b25582d487ab61aa63891e97f75/src/World.cpp?at=master#lines-3935)) is more complex, performing nested iteration of a vector found in the two passed `Rod` computing new positions and updating a neighbour list. It's a similar story

Benchmarking this profiled example before and after the optimisation saw it's runtime reduce from 312 seconds to 31 seconds, a full 10x speedup. Further testing showed that this speedup was widespread across rod models, including under OpenMP. The full scale version of the profiled example now executes in less than 11 hours, down from a full week!

## Conclusion

This work, carried out in August 2025, appears to be the first time FFEA has been systematically profiled. It quickly uncovered two small but significant performance issues, with each one identified and fixed in under two hours.

The first change delivered a modest but meaningful 1.2x speedup for some blob models. Version control showed that this issue had been introduced almost ten years earlier, in November 2015. While a 20 percent improvement may sound unremarkable, at the scale of long running simulations it can translate into hours of saved compute time and faster iteration for researchers.

The second issue had a much larger impact. Fixing it resulted in a roughly 10x speedup for rod based models. This behaviour had been present since February 2018, more than seven years earlier. Notably, the postdoctoral researcher I originally worked with had relied heavily on this code during their PhD, unaware that a substantial fraction of the runtime was being spent on avoidable overhead.

FFEA includes additional solvers and hybrid rod–blob features that were not enabled in these profiles. A multi blob example was also profiled, which showed similar behaviour to the single blob case, although the benefit of the optimisation was reduced due to the extra cost of blob–blob interaction calculations. Many other parts of FFEA have seen little use beyond their original authors, which made it difficult to find realistic examples to profile.

The results raise an uncomfortable but important question. What might have been possible over the past seven years if rod based models had been running ten times faster? Perhaps more ambitious simulations, faster turnaround between ideas, or simply less time spent waiting for long jobs to complete (and reduced HPC utilisation). Even modest investment in profiling can have outsized benefits, especially in long lived research software developed under real world constraints.
