---
level: 1
published: true
authors: Milan Malfait

name: profvis
language: [R]
style: [Function-Level]
website: https://profvis.r-lib.org/
---

[`profvis`](https://profvis.r-lib.org/) is a package to visualise code profiling data from R.
It is the default profiler used when profiling code in the RStudio IDE.

The [Advanced R book](https://adv-r.hadley.nz/perf-measure.html) has a [section dedicated to profiling](https://adv-r.hadley.nz/perf-measure.html#profiling), from which most of the contents below was re-used.


## Quickstart

There are two ways to use profvis:

- From the Profile menu in RStudio.
- With `profvis::profvis()`.

For example, consider the function `f()` below:

```r
library(profvis)
f <- function() {
  pause(0.1)
  g()
  h()
}
g <- function() {
  pause(0.1)
  h()
}
h <- function() {
  pause(0.1)
}

profvis(f())
```

Profiling it with `profvis(f()) will return an [htmlwidget](http://www.htmlwidgets.org/), which will either open in the dedicated RStudio pane or in your browser.

<!-- TODO: add profvis-example.png -->
{% figure caption:"An example of the visualisation provided by profvis." label:profvis-example %}
[![A visualisation of a call-stack's execution timeline within profvis.](/assets/profvis-example.png)](/assets/profvis-example.png) 
{% endfigure %}

The top pane shows the source code, overlaid with bar graphs for memory and execution time for each line of code.
This display gives a good overview of the potential bottlenecks but doesn’t always help you precisely identify the cause. Here, for example, you can see that h() takes 150 ms, twice as long as g(); that’s not because the function is slower, but because it’s called twice as often.

The bottom pane displays a flame graph showing the full call stack. This allows you to see the full sequence of calls leading to each function, allowing you to see that `h()` is called from two different places. In this display, you can hover over individual calls to get more information, and see the corresponding line of source code.

Alternatively, you can use the data tab, which lets you interactively dive into the tree of performance data. This is basically the same display as the flame graph, but it’s more useful when you have very large or deeply nested call stacks because you can choose to interactively zoom into only selected components.