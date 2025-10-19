---
level: 3
published: true
authors: Evgenij Belikov

name: LIKWID
language: [C/C++, Fortran]
style: [Hardware-Metrics]
website: https://github.com/RRZE-HPC/likwid/wiki
---

LIKWID is an open source suite of portable and lightweight command line tools for performance engineering on Linux. It provides, among other functionality, tools for investigating topology of an architecture (`likwid-topology`), for pinning threads and processes (`likwid-pin`), and for measuring hardware performance counters (`likwid-perfctr`).

<!--more-->

## QuickStart

<!-- It would great to provide a full example, with output here. -->

1. Install using info from [https://github.com/RRZE-HPC/likwid/wiki/Build](https://github.com/RRZE-HPC/likwid/wiki/Build). Export PREFIX environment variable (`export PREFIX=/path/to/install/directory`) as the directory where to install LIKWID, then `make` followed by `make install` should suffice to install the tool. You may need to update your `$PATH` environment variable to include the PREFIX path.
2. Check supported performance event groups with `likwid-perfctr -a`, and select one e.g. `FLOPS_DP` or `MEM` to look at performance counter data for floating point performance or memory performance, respectively. 
3. Run `likwid-perfctr -C <pinlist> -g FLOPS_DP ./myapp`. The LIKWID Wiki describes in detail how pinlist, which define where to pin and measure threads, should be specified. E.g. to measure on logical thread 0 of each socket on a hypothetical 2-socket system one would use `-C S0:0@S1:0`.
4. Interpret the results. E.g. is the observed achieved floating operation performance as expected? Note that the retired instructions count may include no-operation instruction. Run time profiling will tell you where the time is spent. Hardware counter measurement will provide relevant low-level metrics that can help explain the behaviour and suggest a suitable optimisation as well as confirm the effectiveness of an optimisation once implemented.

## Additional Information

Please refer to the official [LIKWID Wiki ](https://github.com/RRZE-HPC/likwid/wiki) for detailed examples and in-depth explanation. In particular `likwid-perfctr` also supports [Marker API](https://github.com/RRZE-HPC/likwid/wiki/likwid-perfctr#using-the-marker-api) where instead of measuring for the whole application run you can specify regions of interest to focus on (need to recompile your application). Other modes allow to monitor performance counters on the system regardless of the application for s set duration (stethoscope mode) or to periodically measure at regular intervals (timeline mode).