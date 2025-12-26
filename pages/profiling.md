---
title: Profiling
layout: page
permalink: /profiling/
slug: profiling
---

<!--This content is a cut down version of https://rse.shef.ac.uk/pando-python/profiling-introduction.html-->
This page provides a high-level overview of profiling, for a more detailed introduction read through our long-form [getting started guide](/getting-started/).

If you want to learn more about profiling code in a specific programming language, visit the [profiler database](/profiling/). 

## What is Profiling?

Performance profiling is the process of analysing and measuring the performance of a program or script, to understand where time is being spent during execution. Any code that will run for more than a few minutes over it’s lifetime, that isn’t a quick one-shot script can benefit from profiling.

Profiling is useful when you have written any code that will be running for a substantial period of time. As your code grows in complexity, it becomes increasingly difficult to estimate where time is being spent during execution. Profiling allows you to narrow down where the time is being spent, to identify whether this is of concern or not.

## When to Profile?

Profiling is most relevant to working code, when you have reached a stage that the code works and are considering deploying it. During development, the bottlenecks are likely to change, so profiling at this stage may lead to redundant optimisations.

Profiling should be a relatively quick and inexpensive process. If there are no significant bottlenecks in your code you can quickly be confident that your code is reasonably optimised. If you do identify a concerning bottleneck, further work to optimise your code and reduce the bottleneck could see significant improvements to the performance of your code and hence productivity.

## What to Profile?

The act of profiling your code, collecting additional timing metrics during execution, will cause your program to execute slower. The slowdown is dependent on many variables related to both your code and the granularity of metrics being collected.

Therefore, it is important to select an appropriate test-case that is both representative of a typical workload and small enough that it can be quickly iterated. Ideally, it should take no more than a few minutes to run the profiled test-case from start to finish, however you may have circumstances where something that short is not possible.

For example, you may have a model which normally simulates a year in hourly timesteps. It would be appropriate to begin by profiling the simulation of a single day. If the model scales over time, such as due to population growth, it may be pertinent to profile a single day later into a simulation if the model can be resumed or configured. A larger population is likely to amplify any bottlenecks that scale with the population, making them easier to identify.

## Profiling Styles

There are several different styles of profiling, which are common between profiling tools across programming languages.

- [Manual Profiling](#manual-profiling)
- [Function-Level Profiling](#function-level-profiling)
- [Line-Level Profiling](#line-level-profiling)
- [Timeline Profiling](#timeline-profiling)
- [Hardware Metric Profiling](#hardware-metric-profiling)
- [Memory Profiling](#memory-profiling)

### Manual Profiling

Similar to debugging with print statements, manually timing sections of your code is a rudimentary form of profiling.

Whilst this can be appropriate for profiling narrow sections of code, it becomes increasingly impractical as a project grows in size and complexity. Furthermore, it’s also unproductive to be routinely adding and removing these small changes if they interfere with the required outputs of a project.

You may have used this approach before to ensure reasonable performance, but there are better methods!

### Function-Level Profiling

Software is typically comprised of a hierarchy of function calls, both functions written by the developer and those used from the language’s standard library and third party packages.

Function-level profiling analyses where time is being spent with respect to functions. Typically function-level profiling will the total time spent executing each function, inclusive and exclusive of child function calls. <a href="#deterministic-vs-sampling-profilers">Deterministic</a> profiling tools may also count the number of times each function is called. 

This allows functions that occupy a disproportionate amount of the total runtime to be quickly identified and investigated.

Function-level profiling will normally be your first choice when profiling to achieve reasonable performance, as it can quickly expose the most computationally expensive functions.

### Line-Level Profiling

Function-level profiling may not always be granular enough, perhaps your software is a single long script, or function-level profiling highlighted a particularly complex function.

Line-level profiling provides greater granularity, analysing where time is being spent with respect to individual lines of code. Some profiling tools will line profile a whole codebase, others will require you to specify the functions to be line profiled.

This will identify individual lines of code that occupy an disproportionate amount of the total runtime.

Line-level profiling may be useful to achieve reasonable performance, if identifying the slow function didn't provide enough information to understand the bottleneck.

### Timeline Profiling

Timeline profiling takes a different approach to visualising where time is being spent during execution.

Typically a subset of function-level profiling, the execution of the profiled software is instead presented as a timeline highlighting the order of function execution in addition to the time spent in each individual function call. This can be particularly useful for parallel code, as idle threads and synchronisation points are clearly exposed.

By highlighting individual functions calls, patterns relating to how performance scales over time can be identified. These would be hidden with the aggregate function-level and line-level approaches.

In most cases you're unlikely to require timeline profiling to ensure reasonable performance.

### Memory Profiling

Memory profiling instead profiles the allocation and release of memory, this can be useful for identifying performance issues related to high memory consumption such as memory leaks.

If your code consumes too much memory, running it through a memory profiler might help explain the cause.

### Hardware Metric Profiling

Processor manufacturers typically release advanced profilers specific to their hardware with access to internal hardware metrics. These profilers can provide analysis of performance relative to theoretical hardware maximums (e.g. memory bandwidth or operations per second) and detail the utilisation of specific hardware features and operations.

Using these advanced hardware metrics requires a thorough understanding of the relevant processor architecture and may lead to hardware specific optimisations.

Examples of these profilers include; Intel’s VTune, AMD’s uProf, and NVIDIA’s Nsight Compute.

You shouldn't need access to hardware metrics to ensure reasonable performance.

## Deterministic vs Sampling Profilers

There are two main approaches that profilers use for function and line level profiling, deterministic and sampling. The majority of timeline, memory and hardware metric profiling is implemented in a deterministic manner.

Deterministic profilers provide precise measurements by recording every relevant event (e.g. function or line executed) during execution. This comprehensive tracking ensures consistent and reproducible results, however due to this granularity deterministic profiling can significantly slow down program execution and generate large volumes of data. For this reason deterministic profilers are best suited for small profiles.

In contrast, sampling profilers operate similar to a debugger, pausing the profiled code at a regular sampling interval (e.g. 1000 times a second). The profiler then logs where the code was paused before resuming execution. This approach is significantly less granular in it's data collection, however any component that occupies a significant proportion of the runtime will inevitably be caught by a significant proportion of the samples. This approach is low-overhead, not significantly increasing the runtime during profiling. Sampling profilers typically allow the sampling rate to be increased, to increase collection granularity, this does however increase the overhead of profiling. Advanced profiling tools typically utilise sampling (atleast for a subset of their metrics), so that they can be used to profile large and complex software.

