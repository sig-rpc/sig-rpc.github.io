---
title: Parallel
layout: page
permalink: /parallel/
---

If you've profiled your code and reached a point of reasonable performance, you may want to consider whether it can be sped up further with parallelism. There are several different forms of parallelism that can be used, however each has different trade-offs which may make them unsuitable for your code, or too challenging to utilise without advanced skills.

<!--When considering parallelisation, it's important to consider Amdahls law, that "the overall performance improvement gained by optimizing a single part of a system is limited by the fraction of time that the improved part is actually used. In terms of parallelisation, this means that if only the code behind 50% of our runtime can be parallelised, we could at best expect a 2x speedup.-->

The following parallel forms are outlined below:

* [Vectorisation](#vectorisation)
* [Multi-threading](#multi-threading)
* [GPU](#gpu)
* [Distributed (HPC)](#distributed-hpc)

It's recommended that you read all four sections, before deciding which to pursue.

## Vectorisation

<!-- What -->
Vectorisation is a form of "Single Instruction Multiple Data" (SIMD) parallelism, whereby special vector instructions perform the same operation on multiple numbers simultaneously. To achieve this, there needs to be enough data to fill the processor's vector registers (e.g. Intel AVX(2)'s are 256bit, either 8 `float`, or 4 `double`), stored in contiguous buffers and the compiled code needs to call the appropriate vectorisation instructions.

<!-- Why -->
As long as data is formatted correctly there's no overhead to utilising vectorisation, so vectorising even a single instruction should improve performance. Likewise, the simplicity of vectorisation can lead to the parallel speed-up of the vectorised code being very close to linear. This kind of parallelism can greatly improve the performance of large numeric codes, where mathematical operations are applied across large buffers and data-tables. 

<!-- How (Not Recommended) -->
Modern C/C++ compilers try to auto-vectorise appropriate code, but they're far from fool-proof, frequently missing opportunities (different compilers may even find different auto-vectorisation opportunities). If you're writing C/C++ it is possible to explicitly call vectorisation directly with assembly or `std::simd`, but this isn't encouraged for typical users.

<!-- How (Direct) -->
Instead, if you're using OpenMP you can take advantage of `#pragma OMP simd`. When applied to appropriate `for` loops, it provides a strong hint to auto-vectorise and may provide compilation warnings where it fails.

<!-- How (In-direct) -->
More broadly, if you're using high-performance mathematical libraries (such as NumPy, LAPACK, BLAS etc) you're probably already taking advantage of vectorisation. The developers of high-performance libraries do all the hard work in ensuring vector instructions are used, you just need to check the relevant library's documentation to ensure you're following their best practices.

## Multi-threading

<!-- What -->
Modern processors have multiple-core architectures, each processor core is able to operate independently. Early multi-core processors were only dual-core, but now processors typically have 8 or 16 cores (even as high as 96 in some cases). However all cores are not equal, their performance can differ substantially. A modern processor may have; performance (p-cores), efficient (e-cores) and logical cores (hyper-threading/simultaneous multi-threading).

<!--
P-cores and e-cores are Intel's latest innovation. P-cores are intended for high performance workloads. In contrast e-cores are optimised to be energy efficient, offering less absolute performance than p-cores. At runtime, the operating system and processor decide which cores software is given access to, so users and software developers have little control over the impact. AMD is likely to follow suit with a similar e-core implementation in future.

Logical cores have been around longer, these are a representation of core's support for Intel's hyper-threading or AMD's simultaneous multi-threading (SMT). Each logical core, is essentially a physical core's support for hyper-threading/SMT. This distinction is required because logical cores are not able to achieve the performance of physical cores. While hyper-threading/SMT will normally improve the instruction throughput, it will also often lead to reduced performance of any single thread.
-->

<!-- Why (Limitations) -->
It is very difficult to achieve a linear parallel speed-up with CPU multi-threading, even if logical and e-cores are ignored. CPUs have limited hardware resources such as caches, it is rare for parallel workloads to be balanced such that hardware bottlenecks are not reached. Furthermore, increased activity leads to increased thermal output which may increase thermal throttling of the processor.

<!-- How (Caveat) -->
Regardless if you wish to utilise multi-threading the rule for data storage is the opposite to that for vectorisation. Parallel threads should operate on variables that are stored in different cache lines. When processors load memory, they load a full 64-byte line from RAM into their cache, rather than individual 4-byte floats. That cache line is then copied into a processor core's smaller local cache. If multiple cores are operating on the same cache line in parallel, they will keep invalidating each other's copy, forcing otherwise redundant reloads. This can greatly impact performance. Vectorisation can be combined with multi-threading, the cache-line and vector buffer lengths are typically compatible.

<!-- Todo row-wise vs column-wise parallel matrix operation example? -->
<!-- How -->
The easiest way to take advantage of multi-threading is via packages such as [OpenMP](https://www.openmp.org/) (C/C++/Fortran), [multiprocessing](https://docs.python.org/3/library/multiprocessing.html) (Python) and [parallel](https://r-universe.dev/manuals/parallel.html) (R). Furthermore, modern programming languages increasingly offer simple drop-in replacements to parallelise tasks such as `for` loops and other operations on arrays/data-tables (it can be worth checking documentation).

These abstractions reduce the need to thoroughly understand threads and other parallel concepts. Instead they simply requiring you to decide whether your workload is data parallel (e.g. an operation applied to all elements of an array or data-table, often in the form of a `for` loop) or task parallel (e.g fully independent jobs) and to identify race-conditions (where two threads operate on the same data). Small amounts of restructuring or additional code can then introduce multi-threading.

Further to the above reasons, performance of multi-threading via these methods is further limited by the overhead of creating and destroying parallel environments and how loads are balanced between threads. In order to maximise performance, again the relevant package's documentation should be reviewed to ensure best practices are followed.

There are many common pitfalls, such as [false sharing](/optimisation/openmp/false-sharing), which can lead to a disappointing speedup when OpenMP is added to a codebase. Make sure to review our [OpenMP optimisation database](/optimisations/openmp) to avoid them!

## GPU

<!-- What -->
GPU parallelism requires distinct accelerator hardware suitable for some highly parallel tasks, in contrast to a CPU's upto 100 parallel cores, GPUs require 3 order of magnitude more threads to be fully saturated (e.g. 100,000 threads).

<!-- Background -->
<!--GPUs first came about as specialist hardware for rendering computer graphics. A scene contains models, each made up of many thousands of vertices (points in space) and polygons (collections of 3+ vertices that form a flat surface). To convert these to an image, each vertex and polygon fragment must pass through the same vertex and fragment shading pipelines, convert them from model space to screen space, handling overlapping elements and calculating colours. This lead to the development of specialist graphics processors, to achieve the performance required for real-time rendering. Graphics processors evolved, first with a programmable graphics pipeline and later dedicated general purpose GPU (GPGPU) frameworks such as CUDA and OpenCL.-->

<!-- Why/When -->
GPUs have much in common with CPUs vector instructions, however GPUs operate slower than CPUs and must be driven by an application running on a CPU. With the exception of NVIDIA's GraceHopper superchip, GPUs do not share memory with the main CPU, this adds an additional latency when copying data to or from a GPU. Despite these drawbacks, due to the massive parallel scale GPUs operate at they will often outperform CPUs with suitably large parallel workloads.

<!-- How (Limitations) -->
Most GPU parallel code to-date has been written using NVIDIA's proprietary CUDA framework. NVIDIAs early entry to the field of GPGPU, with over a decade of limited competition led to all substantial GPU parallel libraries being built using CUDA. This is now gradually changing, AMD (and Intel) have both entered the accelerated compute market, and increasingly code is being written using AMDs ROCM and HIP, and SyCL (the spiritual successor to OpenCL). However it remains that most GPU parallel code is specific to a particular brand's GPUs, so it's worth considering which GPUs you have access to before progressing further.

<!-- How -->
Writing efficient code to target GPUs directly is challenging. GPUs are a specialist architecture, without a thorough understanding of this architecture there are many pitfalls that will harm performance. Whilst code targetting GPUs is written in C++, much of the C++ standard library such as all the common data structures is unavailable. Along with the need to manually transfer data to the GPUs memory, this can lead to any non trivial code to require significant rewrites to appropriately utilise GPU compute.

Similar to OpenMP for multithreading, [OpenACC](https://www.openacc.org/) and modern [OpenMP's offload functionality](https://www.openmp.org/wp-content/uploads/2021-10-20-Webinar-OpenMP-Offload-Programming-Introduction.pdf) (OpenMP 4.0+) can be used to convert existing C/C++/Fortran code to utilise GPU compute. Due to the complexity of GPU compute there are additional considerations to be managed when using these, such as data movement. If data is regularly copied back and forth between CPU and GPU with only a small amount of computation on the GPU, the data movement overhead will quickly exceed time spent performing compute.

Many libraries also provide abstracted access to GPU compute. NVIDIA provides [`cuPyNumeric`](https://docs.nvidia.com/cupynumeric/latest/)and [`cuDF.pandas`](https://docs.rapids.ai/api/cudf/stable/), drop-in replacements for the Python packages NumPy and Pandas. Other scientific libraries

## Distributed (HPC)

<!-- What -->
Distributed computing is where compute is distributed over multiple nodes (computers). This is typically available via HPC clusters, although also historically utilised volunteers spare compute in the form of SETI@Home and Folding@Home.

There are two common approaches for utilising distributed compute, Message Passing Interface (MPI) and job or task arrays

MPI is an advanced API for C/C++/Fortran programmers that allows programmers to distribute workloads across multiple processes, whether they are in the same computer or distributed over a network. You're unlikely to want to write MPI code directly, however it's possible if you're using scientific compute packages that they have MPI support. HPC systems that support multi-node jobs, should provide an MPI implementation such as OpenMPI or MPICH that can be used.

In contrast job arrays (SLURM terminology) are a straight forward technique to split up a HPC job that contains multiple independent units, so that they can be executed in parallel across a cluster. The maximum number of tasks one can be split into will vary depending on the HPC system, it could be anywhere from 50 to 1000. It's recommended to ensure that each individual task is still a substantial workload, otherwise the scheduling cost will outweigh that of the job itself.

A SLURM task array is launched via `--array=0-31` which would submit a job 32 times, with ID's `[0-31]` (inclusive-inclusive). Within the job script it is necessary to use the tasks's unique ID to configure it's specific task, this could be as simple as using it's ID as an index into a list of input configurations.

SLURM will provide the job script with 6 relevant variables at execution:

```
# The common job ID shared by all tasks in the array
SLURM_JOB_ID
# The unique per task job ID
SLURM_ARRAY_JOB_ID
# The unique ID of the task, according to what was passed via --array
SLURM_ARRAY_TASK_ID
# The total number of tasks
SLURM_ARRAY_TASK_COUNT
# The maximum unique task ID
SLURM_ARRAY_TASK_MAX
# The minimum unique task ID
SLURM_ARRAY_TASK_MIN
```

The simplest example of a task array job script would be:

```
#!/bin/bash
#SBATCH ...           # Usual SLURM job configuration
#SBATCH --array=0-31  # Array range


# Run my program using SLURM_ARRAY_TASK_ID
./my_program -i inputs/${SLURM_ARRAY_TASK_ID}
```

This could be made more advanced, processing multiple jobs per task, by dynamically dividing a for loop within the job script.

Tasks arrays are a really easy way to speed up the execution of embarrassingly parallel tasks, where you have many independent jobs to be executed. It's worth reviewing the documentation for the HPC system that you will be using, exact usage may vary slightly depending on how the scheduler is configured.

#### Profiling slurm job efficiencies

When you submit a job to slurm, it always requests a certain amount of compute resources (such as CPU cores and memory). These will either come from the HPC system's defaults or from the values you specify in your job script.

This gives scope to introduce inefficiencies by over-specifying or under-specifying the memory and CPU/GPU resources for each job. Over specifying resources implies that the resources are allocated but not used - depriving other jobs and ultimately slowing the HPC system down; whereas under specifying resources can increase the time taken to run the job, or even cause it to fail if the allocated memory is too small.

A good first step in identifying inefficient slurm resource allocations is to run the program `seff` (short for 'slurm efficiencies') on a completed job, passing the job ID. This gives several useful measurements, e.g.:


```
seff 12345678
Job ID: 12345678
Array Job ID: 12345678_11
Cluster: my_hpc
User/Group: a.user/a.group
State: COMPLETED (exit code 0)
Cores: 20
CPU Utilized: 37-12:06:38
CPU Efficiency: 99.42% of 37-17:21:04 core-walltime
Job Wall-clock time: 14:08:46
Memory Utilized: 9.96 GB
Memory Efficiency: 3.98% of 250.00 GB

```

In this example, the CPU efficiency is high at over 90%, but the memory used is very low - so reducing the allocated memory may be more efficent to potentially allow more jobs to simultaneously run, depending on the number of cores on a node. 

