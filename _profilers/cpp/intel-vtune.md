---
level: 2
published: true
authors: Robert Chisholm

name: Intel VTune
language: [C/C++, OpenMP, Python]
style: [Function-Level, Line-Level, Memory, Hardware-Metrics]
website: https://www.intel.com/content/www/us/en/developer/tools/oneapi/vtune-profiler.html
---

Intel VTune Profiler is a fully-featured profiler that supports a broad range of languages. Users wishing to perform function or line level profiling can use the default "Hotspots" configuration, that will output profiling results in an interactive GUI with several visualisation options to aide interpretation of CPU time within the profiled application.

There are also many more advanced profiling configurations available, covering memory and hardware metrics, these are unlikely to be useful for the typical programmer.

<!--more-->

*Refer to the [Python](#python) section below to learn about profiling Python with Intel VTune.*

## QuickStart (GUI)

**1.** Install Intel VTune

Intel VTune profiler is freely available, it can be installed by following Intel's [installation guide](https://www.intel.com/content/www/us/en/docs/vtune-profiler/installation-guide/2025-0/overview.html). Be sure to select the latest profiler version.

**2.** Compile your program with both release optimisations and debug symbols.

If using a compiled language, in order for the profiler to include symbol information (so the profiler will list your functions by their names) your code needs to be compiled with debug information. However, you should also include release optimisations to ensure the profiled performance is representative.

If using CMake this could be as simple as using the pre-defined build configuration `RelWithDebinfo`, if the project is configured to support it.

```sh
# Re-configure in-source build as release optimised with debug symbols configuration
cmake .. -DCMAKE_BUILD_TYPE=RelWithDebinfo
# Rebuild the project with 4 threads
cmake --build . --parallel 4
```

If calling your compiler directly, you are likely to need to explicitly pass both `-g` for debug symbols and `-O3` to maintain optimisations. These are common to most C/C++ compilers.

**3.** Profile your code

Open Intel VTune via your operating system's start menu or `./vtune-gui`.

By default it will show the welcome tab, click the large "Configure Analysis..." button, or select the configure analysis option (the <i class="bi-play" title="play"></i> icon) from the left-hand menu.

{% figure caption:"Intel VTune's configure analysis tab." label:configure-analysis %}
![A screenshot of Intel VTune's configure analysis tab showing three panes; where, what and how. The Where pane has an icon of a laptop with the option selected as local host. The What pane has an icon of a clapperboard with the option selected as launch application, it provides sub fields for configuring the application, the application's parameters and working directory. The How pane has an icon of a flame with the option selected as hotspots, the it provides sub options to switch between user-mode sampling (selected) and event-based sampling with a minimised details block.](/assets/intel_vtune/configure_analysis.png) 
{% endfigure %}

The default settings, shown above, to profile on the "Local Host", via "Launch Application" with "Hotspots" profiling using "user-mode sampling" are sufficient for function-level and line-level profiling. You only need to complete the application field, by selecting the location to the program to be profiled. If necessary, you should also specify any parameters passed to the application ("application parameters") and it's "working directory".

Once configured, you can click the start (<i class="bi-play" title="play"></i>) to begin profiling. As shown below, Intel VTune displays the application's output log and the elapsed execution time whilst collecting profiling data. The stop button (<i class="bi-stop" title="stop"></i>) can be pressed, to exit the application early and display the currently collected profiling data.

{% figure caption:"Intel VTune's collection log tab, whilst the profiling data is being collected." label:opening-result %}
![A screenshot of Intel VTune's collection log tab whilst profiling data is being collected. It includes a console titled application output, which shows the applications output stream. Below this there are buttons to start, pause, stop and cancel profiling.](/assets/intel_vtune/opening_result.png)
{% endfigure %}

Once complete, the profiling results will be opened.

## Interpreting Output

On opening hotspots profiling results the summary tab will be shown, with other tabs "Bottom-up", "Caller/Callee", "Top-down Tree" and "Flame Graph" which provide different visualisations of the function-level profiling results.

{% figure caption:"Intel VTune's hotspots profile Summary tab." label:summary %}
![A screenshot of Intel VTune's results summary for a hotspots profile.](/assets/intel_vtune/summary.png)
{% endfigure %}

The hotspots profile summary shows some high-level information, most important of these is the "Top Hotspots" section which lists the functions that consumed the most runtime.

By default, the **Bottom-up** tab lists functions in order of CPU time (grouped by Function/Call Stack), they can then be expanded to show the call stacks in which they were called. This can be useful if an expensive function is called in multiple places, to identify the most expensive.

{% figure caption:"Intel VTune's hotspots profile Bottom-up tab." label:bottom-up %}
![A screenshot of Intel VTune's results bottom-up results for a hotspots profile.](/assets/intel_vtune/bottom_up.png)
{% endfigure %}

By default, the **Caller/Callee** tab lists functions in order of their total CPU time, inclusive of child function calls, displaying self CPU time (without child function calls) in the neighbouring column. When a function is selected, it's callers and callees are displayed with their respective CPU times relative to the selected function's CPU time in panels on the right-hand side.

{% figure caption:"Intel VTune's hotspots profile Caller/Callee tab." label:caller-callee %}
![A screenshot of Intel VTune's results caller/callee results for a hotspots profile.](/assets/intel_vtune/caller_callee.png)
{% endfigure %}

By default, the **Top-down Tree** tab provides an inverse to the Bottom-up tab, a tree starting from the root function call can be expanded to visualise paths through the call stack and respective CPU times.

{% figure caption:"Intel VTune's hotspots profile Top-down tree tab." label:top-down-tree %}
![A screenshot of Intel VTune's results top-down tree results for a hotspots profile.](/assets/intel_vtune/top_down_tree.png)
{% endfigure %}

Finally, the **Flame Graph** tab provides an interactive visualisation to the Bottom-up results (it can be toggled to Icicle graph to display top-down results). The width of the graph represents the total CPU time of the application, each layer above this shows child function calls and with widths relative to their respective CPU times. Hovering a function's box displays it's information in a tooltip, and clicking one zoom's the flame graph such that the selected function becomes the root with full width.

{% figure caption:"Intel VTune's hotspots profile Flame Graph tab." label:flame-graph %}
![A screenshot of Intel VTune's results flame graph for a hotspots profile.](/assets/intel_vtune/flame_graph.png)
{% endfigure %}

**Line-level** profile information can be accessed for functions, if the source file has not changed, by right clicking them and selecting "view source" from the context menu. This will open the relevant function's source file, and focus the most expensive line of code.

{% figure caption:"Line-level profiling information shown within Intel VTune after selecting to view a function's source." label:line-level %}
![A screenshot of line-level profiling information within Intel VTune.](/assets/intel_vtune/line-level.png)
{% endfigure %}

## Python

In order to profile your Python code with Intel VTune you must use the package `pyitt` (which can be installed via `pip`) and create tasks to be profiled within your Python code. For example

```py
import pyitt

with pyitt.task('task_1'):
    # Do something as part of task 1
    # Do something else as part of task 1
    
with pyitt.task('task_2'):
    # Do something as part of task 2
```

Each task will then be displayed as a function within Intel VTune's profiling results.

A [complete guide](https://www.intel.com/content/www/us/en/developer/articles/technical/profiling-data-parallel-python-vtune.html) can be found on Intel's website.

Other Python profilers can operate with little to no changes to your Python code, you may prefer to use those over Intel VTune.

## Limitations

Common to C/C++ profiling, you won't be able to get profiling detail for any libraries used (including the standard library) unless you link against versions with debug symbols.

Intel VTune is a sampling profiler therefore only CPU time is reported by function-level profiling, it does not track how many times each function or line is called.

If profiling code that uses OpenMP, you will find many functions that you don't recognise, some which are only displayed as addresses. These are functions inserted by OpenMP to parallelise your code. Try working with the flame graph visualisation to understand how they fit around the code you have written. You can control some of those functions appearing from the menus in the bottom of the vtune window

{% figure caption:"Intel VTune's menus controlling which functions appear in the various tabs." label:functions-menus %}
![A screenshot of the area in the bottom of the Intel VTune window, showing the menus that control which functions will be included in tables and graphs.](/assets/intel_vtune/functions-menus.png)
{% endfigure %}

For many of the advanced profiling features of Intel VTune it requires administrator access and extra drivers to be installed. Likewise, some hardware metrics available in these advanced profiling modes are only available with a supported Intel CPU.
