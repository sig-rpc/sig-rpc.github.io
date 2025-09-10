---
level: 1
published: true
authors: Robert Chisholm

name: VizTracer
language: [Python]
style: [Timeline]
website: https://viztracer.readthedocs.io/
---

VizTracer is a simple Python package for timeline profiling. It's profiling output can be visualised in a web browser with it's sub-package VizViewer.

<!--more-->

## QuickStart

**1.** `viztracer` can be installed via `pip` or `conda`.

```sh
# install via pip (may not work on MacOS)
pip install viztracer
# or
# install via conda
conda install conda-forge::viztracer
```

**2.** Run the Python code via `viztracer`.

If you would normally execute your script as

```sh
python my_script.py 100
```

You can call `viztracer` with the below, including any required runtime arguments as normal.

```sh
python -m viztracer my_script.py 100
```

**3.** Visualise the output.

On success VizTracer will output a file `result.json` to the working directory. This can then be visualised with `vizviewer`.

```sh
python -m vizviewer result.json
```

## Interpreting Output

VizViewer provides an interactive visualisation of the timeline, that can be panned and zoomed to investigate the call-stack throughout the program's execution.

{% figure caption:"An example of the visualisation provided by VizViewer." label:viztracer-example %}
[![A visualisation of a call-stack's execution timeline within VizTracer.](/assets/viztracer-example.png)](/assets/viztracer-example.png) 
{% endfigure %}

Within the above example, the profile demonstrates iterative steps of a model on the left-hand side of the timeline followed by generation/export of a plot of the results on the right-hand side where the pattern changes.

The "icicles" on the right-hand side mostly show child function calls within the library MatPlotLib. They are much "taller" because there are significantly more nested function calls than in the code that is part of the model.

Based on their width, it can be seen that each step has roughly the same runtime (e.g. they don't get faster/slower over time), with most of the step time occupied by 1 sub-function (shown in purple/yellow). Zooming into the time-line would expose the name of this function. Likewise, plotting the results takes the same time as 6-7 steps, but most of that time is taken up by expensive MatPlotLib function calls.

## Jupyter Notebooks

If you're more familiar with writing Python inside Jupyter notebooks you can use VizTracer directly from inside notebooks.

**1.** Install `viztracer` and load it's extension.

```py
!pip install viztracer
%load_ext viztracer
```

**2.** Create and fill a `viztracer` magic cell

```py
%%viztracer

# Your code here
```

After the cell has been executed, a "Show VizTracer Report" button will appear, if clicked this will open the results with `vizviewer`.

## Limitations

As a timeline profiler it will collect information for every function call, to avoid running out of memory it uses a circular buffer, whereby [it only holds (by default) the last 1 million function calls](https://viztracer.readthedocs.io/en/latest/basic_usage.html#circular-buffer-size). If your profiled code runs for a very long time, you may lose profiling information from the start of execution.

VizTracer is implemented in Python, so its performance may suffer (unconfirmed) when profiling complex code with many function calls.
