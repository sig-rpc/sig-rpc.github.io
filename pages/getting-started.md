---
title: Getting Started
layout: page
permalink: /getting-started/
---

*This article is approximately 3500 words, and is expected to take 15-20 minutes to read.*

If you're new to learning about performance (and using this website), this long form introduction is a great place to start. The Reasonable Performance Computing SIG website is intended to provide enough introductory software performance material, to get beginners started identifying and (hopefully) resolving performance traps in their own code.

If you're already familiar with the basics, perhaps you want to skip ahead

- [Introduction to Profiling](#introduction-to-profiling)
- [How to Profile](#how-to-profile)
- [Interpreting Results](#interpreting-results)
  - [Flame and Icicle Graphs](#flame-and-icicle-graphs)
  - [Table](#table)
  - [Inline Code](#inline-code)
- [Identifying Optimisations](#identifying-optimisations)
- [Case Studies](#case-studies)
- [Key Points](#key-points)

## Introduction to Profiling

Profiling is the practice of executing code whilst collecting granular metrics, in particular performance profilers collect information to help reveal where time is being spent in the code during execution and why. These range from the time spent executing each function or line of code, to low-level hardware metrics such as achieved memory bandwidth.

Regardless of experience, whether a self-taught or trained programmer, it's always possible to introduce small inefficiencies into your code, that significantly impact performance. These may not impact the correctness of your results, so it's easy for them to go overlooked. Perhaps you added some crude validation code whilst debugging, then months later you've scaled up several orders of magnitude and forgotten about the validation code (as the same bug hasn't resurfaced) but its now accounting for significant proportion of your runtime. Or maybe you're simply unfamiliar with a performant feature of a library or language that you've used.

This kind of gradual, invisible performance degradation is precisely what profiling is designed to uncover by reduce the scope of the code to be optimised from the whole codebase (or a blind assumption), to a small number of lines or functions. As Donald Knuth famously observed, we should only focus our attention on optimising the critical 3% of our code.

> Programmers waste enormous amounts of time thinking about, or worrying about, the speed of noncritical parts of their programs, and these attempts at efficiency actually have a strong negative impact when debugging and maintenance are considered. We should forget about small efficiencies, say about 97% of the time: **premature optimization is the root of all evil**. Yet we should not pass up our opportunities in that critical 3%.

<!-- Could add mathjax to render the formula $O = \frac{1}{(1-P) + \frac{P}{S}}$ -->
To quantify this idea, we can use Amdahl's law which shows that the overall performance improvement from optimising a single part of a system is limited by the fraction of time that the improved part is actually used. [Figure 1](#interactive-amdahls) provides an interactive graph where you can explore changing the the proportion of runtime optimised (`P`).

{% figure caption:"An interactive graph demonstrating Amdahl's law, the overall speedup following a speedup (S) to propotion (P) of the code." label:interactive-amdahls %}
<!-- Interactive Amdahl's law graph, vibe coded with GPT -->
<!-- @todo can I wrap this inside a captioned Jekyll figure? -->
<canvas id="graph" width="600" height="400"></canvas>

<div class="controls">
  <label><tiny>
    Proportion of runtime optimised (P):
    <span id="pValue">0.20</span>
  </tiny></label>
  <input type="range" id="pSlider" min="0" max="100" value="20">
</div>
{% endfigure %}

<!-- Not sure I like this -->
The above graph, clearly demonstrates how theoretically optimising away 20% of the runtime to 0% would provide a 1.25x speedup, showing why we should focus optimisation attention on the significant components. There's little value in optimising away 1% of the runtime if it will have negligible impact to performance.

<!-- Could link to FFEA case-study at final sentence here -->
However Amdahl's law is theoretical, in practice computer hardware is shared and bottlenecks may impact multiple components of the code. Hence, it's possible for some optimisations to improve performance beyond the modified code. A memorable example, saw a profile attribute slightly below 80% of the runtime to redundant object copies, when removed this led to a 10x speedup, yet Amdahl's law would predict a 5x speedup.

You may have previously tried to identify performance bottlenecks by manually timing sections of your code, for example by inserting timing calls around individual functions or blocks. This approach is slow and unwieldy. It is difficult to achieve good coverage even for a moderately sized codebase, and the added instrumentation often requires ongoing effort to update, maintain, and eventually remove as a project evolves.

In contrast, the most basic form of profiling, hot-path profiling (also known as hotspot profiling), requires little additional expertise if you understand your code and can usually be integrated quickly into your release workflow. Profiling may immediately reveal an obvious bottleneck, confirm your assumptions about the most expensive parts of your code, or highlight areas that need further investigation. It is a low-cost exercise that either leads to tangible improvements or gives you greater confidence in the performance of your code.

Most programming languages provide one or more tools for hot-path profiling. While these tools take different forms, the underlying concepts are highly transferable. Once you understand the core principles, you should be able to use them effectively across languages and frameworks.

Hot-path profiling typically operates at two levels, or a combination of both: function-level and line-level. Function-level profiling measures metrics for each function, including time spent both inclusive and exclusive of child function calls. Line-level profiling instead tracks execution time for each line of code. Although line-level profiling generates much more data, it can provide additional context when function-level metrics alone are insufficient.

Hot-path profiling tools generally use one of two approaches to collect performance data. The simplest are deterministic profilers, which record every function call and every executed line of code. This method provides detailed insights, such as how many times each function is called or how often each branch of a conditional is executed, but it can be computationally expensive for large codebases.

More advanced profilers typically use a sampling approach. In this method, the profiler periodically checks which line of code is executing, for example 1,000 times per second. The sample rate is usually configurable. Sampling captures only a fraction of the data collected by deterministic profilers, so it provides an approximate picture. However, this approximation is sufficient for our purposes. We are primarily interested in the most expensive parts of the code. If a function consumes a significant portion of runtime, for example 10 percent or more, it will normally appear proportionally in the samples. Sampling profilers therefore introduce much lower overhead, making them ideal for profiling expensive code.

It is not essential to memorise these differences. In most cases, you can simply use the profiler that is available. The key points are summarised in [figure 2](#profiler-type-table) below.

{% figure caption:"A table summarsing the strengths and weaknesses of different profiling tools." label:profiler-type-table %}
| Collection method | Attribution level | Strengths                           | Weaknesses       |
| ----------------- | ----------------- | ----------------------------------- | ----------------- |
| Sampling          | Function          | Minimal overhead                    | Statistical noise for small runs |
| Sampling          | Line              | Low overhead, fine-grained insight  | Potentially biased by timing  |
| Deterministic     | Function          | Exact call counts, timings          | Moderate overhead |
| Deterministic     | Line              | Precise attribution, branching info | High overhead     |
{% endfigure %}

In addition to common hot-path profiling tools, you may come across more specialist profilers. In general these take two forms.

Firstly there are timeline profilers, these typically operate similar to function-level profilers however instead of presenting aggregations of the function-level metrics, they present function calls on a timeline of the code's execution. Timeline profiling is particularly useful when investigating parallel codes, as it clearly displays synchronisation points and idle threads. It may also be useful if profiling a code, where performance degrades over time as this can be difficult to diagnose with aggregated metrics.

You may also come across advanced profiling tools, which we consider "hardware metric" profilers. These are typically provided by hardware manufacturers (e.g. Intel VTune, AMD μProf & NVIDIA NSight Compute) and gather metrics from hardware-specific counters. They allow expert users to analyse data such as theoretical vs achieved memory bandwidth or floating-point operations per second. Interpreting these metrics correctly requires deeper knowledge, so it’s normal if they don’t immediately make sense. For most purposes, you can focus on hot-path profiling. Some of these tools, including Intel VTune, also offer a hot-path profiling mode, which simply needs to be selected when starting a profiling session.

In addition to performance profiling, profiling can sometimes refer to memory profiling, these tools are designed for tracking memory allocations to help diagnose the source of memory leaks (when a program's memory utilisation erroneously continues increasing) or merely high memory utilisation. While memory profilers are valuable for diagnosing memory issues, they are generally not helpful for evaluating overall software performance.

Likewise, if your code is doing something more advanced such as distributed (e.g. MPI) or GPU (e.g. CUDA), the remaining guidance is unlikely to be of great help. You may find it useful to hot-path profile a serial version of the code, but in both these cases communication overhead can become the largest performance bottleneck which may not be well reflected in traditional profiling tools. Consider instead reaching out to HPC communities, regional and national HPC centres often host events and training related to both writing and optimising distributed and GPU codes. 

## How to Profile

<!-- When to profile -->
The best time to profile code is once development is complete and before it is put into use or published. As a codebase evolves, performance bottlenecks often shift. Code that accounts for a large fraction of runtime early on may become negligible later as features are added or implementations change.

During development, testing is frequently performed on small data sets or populations. Algorithms with poor scaling behaviour may therefore appear acceptable until they are exercised at production scale, sometimes several orders of magnitude larger.

Although profiling unfinished code is not inherently wrong, it can be counterproductive. It risks diverting attention from core development, and optimising code that is still in flux can introduce subtle bugs that are hard to detect and any speedup may be rendered irrelevant by subsequent changes.

<!-- Select workflow to profile -->
When your code is finished, decide which workflow you want to profile. The results are only meaningful for code paths that are actually executed, so the selected workflow should closely resemble the one you intend to optimise.

For highly configurable applications, it is usually best to choose a smaller or simplified configuration that is still representative of typical behaviour. This keeps profiling overhead manageable while still exercising the relevant performance-critical paths.

If the code is iterative, such as a simulation that runs for thousands of steps, profiling the full execution is often unnecessary. Since each iteration typically executes the same code, profiling only the initial steps will usually produce similar insights at a much lower cost, aside from any one-time initialisation overhead.

Be aware that profiling adds runtime overhead and will slow execution. To minimise time spent profiling, it is generally preferable to profile a configuration that runs for several minutes. If this is not feasible, profiling a longer or full configuration is still acceptable. In that case, note that some profilers stop collecting data once internal buffers are full, although if you run into this behaviour it can often be adjusted by configuring buffer sizes or delaying the point at which the profiler begins to collect data.

<!-- Configuring the code to be profiled -->
Most profilers can be used without modifying your source code. The main exception is compiled languages such as C and C++, which typically require the build to be configured explicitly for profiling.

This usually involves compiling with both debug symbols and compiler optimisations enabled. Debug symbols allow the profiler to map the compiled machine instructions back to locations in the source code, while compiler optimisations ensure that the profiled execution is representative of production.

Some profilers may also require limited changes to the code, such as explicitly marking functions to be profiled or controlling when data collection begins and ends. These requirements vary between tools and use-case, so you should consult the documentation for your chosen profiler to determine whether any configuration or code changes are necessary.

## Interpreting Results

Once your code executes to completion (without crashing) via the profiler, you should be able to view the results. There are a few common approaches in which profilers represent these results:

### Flame and Icicle Graphs

Flame graphs and icicle graphs are standard visualisations for aggregated, function-level profiling data. Flame graphs place the root at the bottom and grow upwards, while icicle graphs place the root at the top and grow downwards. Aside from orientation, they are interpreted in exactly the same way.

The root row always contains a single cell with 100 percent width, representing the entry point of the profiled execution, typically the main function. Above or below it, depending on the orientation, successive rows show cells corresponding to functions called by their parent. The width of each cell represents the aggregated runtime spent in that function, including time spent in its callees.

Child cells rarely occupy the full width of their parent. Most functions spend time performing work other than calling child functions, such as control flow, arithmetic, or data manipulation. This unused space produces the stepped appearance characteristic of flame ([Figure 3](#flame-graph)) and icicle ([Figure 4](#icicle-graph)) graphs, which gives them their names.

These graphs are typically interactive. Selecting a cell redraws the graph with that function as the new root, making it easier to inspect deeper call hierarchies that may otherwise be difficult to see.

Overall, flame and icicle graphs provide a concise visual overview of how execution time is distributed across function call hierarchies within a profiled program.

<!-- Would be nice to group these figure's in a combined sub-figure or similar -->
{% figure caption:"Intel VTune's hotspots profile Flame Graph tab." label:flame-graph %}
![A screenshot of Intel VTune's results flame graph for a hotspots profile.](/assets/intel_vtune/flame_graph.png)
{% endfigure %}
{% figure caption:" An example of the default 'icicle' visualisation provided by `snakeviz`." label:icicle-graph %}
![A web page, with a central diagram representing a call-stack, with the root at the top and the horizontal axis representing the duration of each call. Below this diagram is the top of a table detailing the statistics of individual methods.](/assets/snakeviz-example.png)
{% endfigure %}

### Table

Another common way to present function-level profiling data is as a table, often referred to as a profile summary. This lists functions alongside their measured runtime, typically reported both inclusive and exclusive of time spent in child function calls. Some profilers also record how many times each function is invoked, allowing a per-call cost to be derived.

These tables provide a fast way to identify the most computationally expensive functions in a program. However, unlike flame or icicle graphs, they do not naturally convey call hierarchy or the context in which functions are executed.

Depending on the tool, the table may be interactive, allowing rows to be sorted or functions to be selected to jump directly to an annotated source code view ([Figure 5](#matlab-summary)). Other profilers provide only a static table representation ([Figure 6](#gprof-table)).

{% figure caption:"An example of the Profile Summary window showing flame graph top, function table bottom. (Screenshot from MATLAB R2025b)" label:matlab-summary %}
![A screenshot of MATLAB R2025b showing the Profile Summary page within the Profiler, which contains a flame graph (top) and a table with function-level profiling results (bottom).](/assets/matlab/profile-summary.png)
{% endfigure %}

{% figure caption:"An example of the Flat profile table output by GProf. (Some names have been compacted)" label:gprof-table %}
```
Flat profile:

Each sample counts as 0.01 seconds.
  %   cumulative   self              self     total           
 time   seconds   seconds    calls   s/call   s/call  name    
 13.75     16.38    16.38 125676749836     0.00     0.00  tVect::operator+(tVect const&) const
 12.60     31.40    15.02 94039864750     0.00     0.00  tVect::operator*(double const&) const
 10.02     43.34    11.94 65081331     0.00     0.00  elmt::correct(double const*, double const*)
  8.54     53.52    10.18 4068178008     0.00     0.00  DEM::particleParticleCollision(particle const*...
  7.62     62.60     9.08 2392019595     0.00     0.00  elmt::predict(double const*, double const*)
  5.66     69.34     6.74 41323211762     0.00     0.00  tVect::operator-(tVect const&) const
  5.45     75.84     6.50 36306445222     0.00     0.00  quaternion::operator*(double const&) const
  4.74     81.49     5.65 7264658795     0.00     0.00  project(tVect, quaternion)
  4.60     86.97     5.48 32022205839     0.00     0.00  quaternion::operator+(quaternion const&) const
  3.61     91.27     4.30  2380119     0.00     0.00  DEM::evaluateForces()
  2.25     93.95     2.68 16726846623     0.00     0.00  tVect::cross(tVect const&) const
  ...
```
{% endfigure %}

### Inline Code

In contrast to function-level visualisations, line-level profiling results are typically presented inline with the source code. Each line is shown alongside profiling metrics in a pseudo-table layout ([Figure 7](#line_profiler-table)), often using visual cues such as background shading to indicate relative cost ([Figure 8](#matlab-function-listing)). Lines with higher execution time are commonly highlighted with darker or warmer colours, making hotspots immediately visible.

This inline representation provides a fast way to identify performance patterns within a function, particularly when scanning large files. It can reveal expensive loops, conditionals, or individual operations that are not obvious from function-level results alone.

For large codebases, which may contain tens or hundreds of thousands of lines, inline views are rarely the primary entry point for analysis. Instead, many profilers require users to navigate from higher-level results, such as function-level summaries, to jump directly to the relevant source locations.

This workflow is intentional rather than limiting. Line-level profiling is most effective once function-level profiling has already narrowed the investigation to a small set of candidate functions. In many cases, function-level results are sufficient on their own, with line-level profiling serving as a deeper diagnostic tool when necessary.

{% figure caption:"An example of the plain-text table output by Python package line_profiler. As an exclusively line-level profiler, it requires users to specify which function they wish to profile. Supporting terminals present line contents with code highlighting." label:line_profiler-table %}
```
Wrote profile results to my_script.py.lprof
Timer unit: 1e-06 s

Total time: 1.65e-05 s
File: my_script.py
Function: is_prime at line 3

Line #      Hits         Time  Per Hit   % Time  Line Contents
==============================================================
     3                                           @profile
     4                                           def is_prime(number):
     5         1          0.4      0.4      2.4      if number < 2:
     6                                                   return False
     7        32          8.4      0.3     50.9      for i in range(2, int(number**0.5) + 1):
     8        31          7.4      0.2     44.8          if number % i == 0:
     9                                                       return False
    10         1          0.3      0.3      1.8      return True
```
{% endfigure %}

{% figure caption:"Part of an example Function listing within a function's profile summary, demonstrating how the most expensive lines appear. Columns, left-to-right, absolute time, calls, line number. (Screenshot from MATLAB R2025b)" label:matlab-function-listing %}
![A screenshot of MATLAB R2025b showing part of a Function listing section of a profile, with two of the lines highlighted with difference shades of red.](/assets/matlab/function-listing.png)
{% endfigure %}

## Identifying Optimisations

Once profiling results have been interpreted, the next step is to determine whether there are obvious optimisation opportunities in the components identified as most expensive.

Many of the easiest performance improvements come from eliminating redundant work. This can take several forms. In some cases the issue is immediately obvious to the code’s author, such as expensive logging, validation, or diagnostics performed on every iteration that could be removed entirely or reduced in frequency. In other cases, profiling highlights expensive calculations or allocation and copy operations that are repeated unnecessarily inside loops and could instead be performed once and reused.

More subtle forms of redundancy are often hidden behind incorrect library usage. For example, passing an inappropriate data type to a library function, such as a [Python list to a NumPy function](/optimisation/python/numpy-types), may trigger an implicit and expensive conversion on every call.

These issues are frequently logical errors rather than complex performance problems, and they can occur within a few lines of code or across several layers of abstraction. Profiling helps focus attention so that these inefficiencies become visible.

Other quick optimisations require more experience to recognise. A common example is using a linear search over a collection such as an array to test for membership. While this approach is intuitive, using a set data structure is often far more appropriate and can be orders of magnitude faster for large inputs.

It is difficult to provide a comprehensive checklist for identifying all possible optimisations. If the most expensive components appear reasonable and no obvious issues stand out, it may be that the code is already performing as well as expected. If doubts remain, there are several practical next steps:

- Look for relevant patterns in the SIG-RPC [optimisation database](/optimisations/) and [external resources list](/resources/), which are regularly updated with new examples.
- Ask a large language model to review a small, self-contained code snippet for performance issues. Providing additional context such as data types and expected input sizes will significantly improve the quality of feedback. Although you should be wary that AI may also provide you incorrect advice if what it tells you doesn't work in practice.
- Request a review from a local Research Software Engineering (RSE) or Research IT team. With profiling results available, experienced developers can often identify potential issues quickly or help investigate less obvious ones.

After applying any optimisation, always re-profile the code and rerun relevant test suites. Performance changes can introduce subtle correctness issues, and removing one bottleneck may expose smaller secondary ones which could be worth optimising too.

A clean profiling report provides additional confidence that your code meets a higher standard of quality, both for running experiments reliably and for publishing reproducible research software. With more experience reviewing your code's performance, you will find it easier to spot similar issues and potentially even catch them whilst writing code.

## Case Studies

We are steadily building a collection of research software profiling case studies. The first published example describes how profiling and optimisation uncovered a tenfold speedup in code that had existed in the codebase for seven years without performance issues being recognised ([read the case study](/blog/casestudy-ffea)).

<!-- Todo where does case-study search exist? -->

If you have had success profiling and optimising your own research software, we would welcome a case study written from your perspective. The more examples we can share, the more clearly we can demonstrate the impact that profiling can have on research code.

If you are interested in contributing, you can [get in touch](mailto:sig-rpc-managers@society-rse.org) to discuss your experience with us, or submit an [issue](https://github.com/sig-rpc/sig-rpc.github.io/issues/new?template=BLANK_ISSUE) or a [pull request](https://github.com/sig-rpc/sig-rpc.github.io/pulls) directly.

## Key Points

The key information to takeaway from this article is:

- Profiling collects detailed performance metrics and highlights where optimisation will have the greatest impact.
- Focus optimisation on code that accounts for a significant portion of runtime.
- Hot-path profilers come in various types, but most are suitable for standard CPU code. GPU or distributed workloads may require specialised tools.
- Profile code once it is functionally complete and ready for release.
- Use a representative workflow or input, ideally one that runs for several minutes.
- Profiling results are typically visualised as flame or icicle graphs, tables showing the most expensive functions/lines, or inline annotations of the source code.
- When a hotspot is identified, examine it for obvious redundancy or inefficiency. Seek guidance from an RSE if unsure.
- After making changes, profile again, as new bottlenecks may appear.

Now that you've read through the introductory materials, you should visit our [profilers database](/profilers/) to find a suitable profiler for your code.

<!--
https://github.com/sig-rpc/sig-rpc.github.io/issues/45
- Purpose of website

- Hello if you're new to performance
 - links to some common questions?

- What is profiling
- Who should profile their code
- Why profile
  - premature optimisation
  - interactive amdahl's law graph
- Doesn't need to be difficult, or time consuming
- Hot path profiling is similar across all languages and profilers
  - function vs line level
  - sampling vs deterministic
  - figure to explain
  - Only need to be able to understand the code you're profiling (and be able to run it)
- Other profiling tools exist, but in most cases you don't need to be concerned by these
  - timeline, good for parallel
  - "hardware metric", good for advanced fine tuning (with relevant technical expertise)
  - memory profiling, maybe you think your code requires more RAM than expected
- If you're code is something more advanced, e.g. MPI (or potentially even some parallel codes), fall outside the scope of this guidance. May benefit from profiling single-threaded variant, however more advanced processes required.
- When to profile
- What to profile
- How to profile (in general)
- Interpreting output
  - Tabular function output (in general)
  - Flame/Icicle graph (in general)
  - Line-level information in general
- Now you've isolated where time is being spent, how to analyse it and spot mistakes
- What next (restart the process)
- Case studies
- Key points
  - Select any "hot path" (e.g. function-level/line-level) profiling to your
  - When your code is ready for production/release, review performance
  - If your results show something suspicious, investigate it further
    - If you manage to resolve this, start again
  - Be happy in the knowledge you've no major performance issues
-->


<script>
  const canvas = document.getElementById("graph");
  const ctx = canvas.getContext("2d");
  const slider = document.getElementById("pSlider");
  const pValueLabel = document.getElementById("pValue");

  const padding = 50;
  const maxSpeedup = 10;

  function amdahlSpeedup(P, S) {
    return 1 / ((1 - P) + (P / S));
  }

  function xFromS(S) {
    const t = 1 - 1 / S;
    return padding + t * (canvas.width - 2 * padding);
  }

  function yFromSpeedup(speedup) {
    return canvas.height - padding -
      (speedup / maxSpeedup) * (canvas.height - 2 * padding);
  }

  function drawAxes() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    drawXGridLines();
    drawYGridLines();

    ctx.strokeStyle = "#000";
    ctx.beginPath();
    ctx.moveTo(padding, padding);
    ctx.lineTo(padding, canvas.height - padding);
    ctx.lineTo(canvas.width - padding, canvas.height - padding);
    ctx.stroke();

    drawXTicks();
    drawYTicks();
  }

  function drawXTicks() {
    ctx.textAlign = "center";
    ctx.textBaseline = "top";

    const ticks = [1, 2, 4, 8, 16, 32, 64, 128, 256, Infinity];
    const minPixelSpacing = 25;

    let lastLabelX = -Infinity;
    const y = canvas.height - padding;

    ticks.forEach(S => {
      let x;
      let label;

      if (S === Infinity) {
        x = canvas.width - padding;
        label = "∞";
      } else {
        x = xFromS(S);
        label = S.toString();
      }

      ctx.beginPath();
      ctx.moveTo(x, y);
      ctx.lineTo(x, y + 5);
      ctx.stroke();

      if (x - lastLabelX >= minPixelSpacing) {
        ctx.fillText(label, x, y + 8);
        lastLabelX = x;
      }
    });

    ctx.fillText(
      "Optimization factor (S → ∞)",
      canvas.width / 2,
      canvas.height - 25
    );
  }

  function drawYTicks() {
    const tickCount = 5;
    ctx.textAlign = "right";
    ctx.textBaseline = "middle";

    for (let i = 0; i <= tickCount; i++) {
      const value = (i / tickCount) * maxSpeedup;
      const y = yFromSpeedup(value);

      ctx.beginPath();
      ctx.moveTo(padding - 5, y);
      ctx.lineTo(padding, y);
      ctx.stroke();

      ctx.fillText(value.toFixed(1), padding - 8, y);
    }

    ctx.save();
    ctx.translate(15, canvas.height / 2);
    ctx.rotate(-Math.PI / 2);
    ctx.textAlign = "center";
    ctx.fillText("Overall speedup (O)", 0, 0);
    ctx.restore();
  }

  function drawXGridLines() {
    const ticks = [1, 2, 4, 8, 16, 32, 64, 128];
    ctx.strokeStyle = "#e0e0e0";

    ticks.forEach(S => {
      const x = xFromS(S);

      ctx.beginPath();
      ctx.moveTo(x, padding);
      ctx.lineTo(x, canvas.height - padding);
      ctx.stroke();
    });
  }

  function drawYGridLines() {
    const tickCount = 5;
    ctx.strokeStyle = "#e0e0e0";

    for (let i = 0; i <= tickCount; i++) {
      const speedup = (i / tickCount) * maxSpeedup;
      const y = yFromSpeedup(speedup);

      ctx.beginPath();
      ctx.moveTo(padding, y);
      ctx.lineTo(canvas.width - padding, y);
      ctx.stroke();
    }
  }


  function drawCurve(P) {
    ctx.strokeStyle = "#0077cc";
    ctx.beginPath();

    let first = true;
    for (let S = 1; S <= 10000; S *= 1.05) {
      const speedup = amdahlSpeedup(P, S);
      const x = xFromS(S);
      const y = yFromSpeedup(speedup);

      if (first) {
        ctx.moveTo(x, y);
        first = false;
      } else {
        ctx.lineTo(x, y);
      }
    }

    ctx.stroke();
  }

  function redraw() {
    const P = slider.value / 100;
    pValueLabel.textContent = P.toFixed(2);

    drawAxes();
    drawCurve(P);
  }

  slider.addEventListener("input", redraw);
  redraw();
</script>
<!-- Hack to enable Kramdown code-blocks inside figure elements -->
<script>
document.addEventListener("DOMContentLoaded", function () {
  document.querySelectorAll("div.highlight").forEach(function (div) {
    if (div.querySelector("pre.highlight")) return;

    const pre = document.createElement("pre");
    pre.className = "highlight";

    while (div.firstChild) {
      pre.appendChild(div.firstChild);
    }

    div.appendChild(pre);
  });
});
</script>
