---
layout: post
title: Welcome to SIG-RPC's Website!
date: 2025-01-12 16:02:20
author: Robert Chisholm (Chair)
categories: website, news
---

Hi and welcome to the reasonable performance computing special interest group's (SIG-RPC's) website. We've put this together over the last couple of weeks based on the ideas discussing during our [second meeting](https://github.com/sig-rpc/minutes/pull/3) (held 2024-12-06).

The website has two goals, firstly to introduce SIG-RPC to the software engineering community, to explain our purpose and mission. Secondly, we hope it can become a home of asynchronous interaction for those interested in sharing and learning about introductory software performance and optimisation.

- [Our Mission](#our-mission)
- [The Website](#the-website)
- [Next Steps](#next-steps)

## Our Mission

As explained on the [home](/) and [about](/about/) pages, SIG-RPC exists to build a community dedicated to the research, development and advocacy of performance best practices for those that work closely with software. We hope to ensure that all code achieves a minimum standard of reasonable performance, whereby all "easy performance wins" have been exhausted.

This came about from my time working as a Research Software Engineer (RSE), with a background in Computer Science, seeing code written by researchers with limited to no formal programming education. It's not uncommon for code to be written with small easily resolved bottlenecks, which cost significant performance. Researchers writing code often lack the understanding to recognise such problems, even if improving performance would benefit their research and productivity. As domain experts, they shouldn't be expected to also be software development experts, but there's no reason we can't make identifying and solving these easy performance wins accessible to them.

This problem isn't however restricted to researchers. There's regular complaints online, that as the performance of hardware has improved software has gotten slower. Within my own Computer Science degree, profiling was not introduced (I now teach an optional academic module that does!) and optimisation was not an explicit focus. I've friends in the private sector that share anecdotes about bad code, written by professional software engineers, that often impacts performance. Likewise, Grand Theft Auto online's loading screen contained two trivial bottlenecks for 7 years before a [gamer identified them and submit a fix](https://nee.lv/2021/02/28/How-I-cut-GTA-Online-loading-times-by-70/).

So the overarching goal of SIG-RPC is simply to get everyone involved in writing software to think more about performance. The goal isn't to teach highly technical low-level optimisations, there's already many High Performance Computing (HPC) communities for that. But to encourage people to profile their code, once it's ready, and check that the performance profile makes sense. This shouldn't be time consuming where code already meets a reasonable standard of performance, it just requires someone to understand how to profile the code and take an hour to do so. In the case you do find something, fixing that could potentially save hundreds of hours of runtime over the lifetime of the code. So it's a worthwhile trade.

Faster code means greater productivity and reduced energy usage, with potential secondary effects of improved mental health, and identifying and resolving many such bottlenecks can be accessible to programmers of all skill levels (maybe not total beginners) without requiring full academic sized modules.

## The Website

Based on discussions at our second meeting, the website has four main areas into which information is divided. It's hoped that over time members of the community will contribute further information to help grow each of these areas [via GitHub](https://github.com/sig-rpc/sig-rpc.github.io/issues/new/choose).

### [Profiling/Profiler Database](/profiling/)

This area of the website provides a brief introduction to the concept of profiling, followed by providing a database of quick-start guides for many different profilers, of differing styles (e.g. function-level) for different languages.

Once you know the basics of a style of profiling, skills are relatively transferable between profilers and languages. For this reason, documentation for profilers can sometimes be unhelpful for newcomers. The hope is that this area of the website can help you find an appropriate profilers, give you a quick example of usage and notify you of any known issues or limitations.

### [Optimisation Database](/optimisations/)

Similar to the profiler database, this area is a database of optimisation patterns. Once you've profiled your code and identified a bottleneck, the hope is that if you don't know how to address it (or whether it's reasonable) that you can find a similar optimisation pattern in the database to follow.

Some of these optimisations will be about correct data-structure and algorithm usage, computer science theory, common to most programming languages. Whereas others are language or even library specific, potentially incidental finds which are not obvious without investigation. For this reason, it's really important that people begin to look out for simple optimisation patterns, so our database can grow for the collective benefit.

### [Parallel](/parallel/)

Once code meets a reasonable standard of performance, if further performance is required it's likely the next step is to consider parallelisation. Whilst outside the scope of the reasonable performance community, this area provides a high level guide to help you understand what might be appropriate for your case and where to go next.

### [Resources](/resources/)

Finally, the resources page provides further reading if you'd like to go deeper into the topics covered by this website. It's a collection of related training, books, websites and more.

## Next Steps

Now that the initial website has been released, there's still a long way to go building up the website. I personally have a backlog of optimisations to write up and add to the database. If you have a favourite profiler, or optimisation pattern you'd like to submit please to submit an issue or pull request on [our GitHub](https://github.com/sig-rpc/sig-rpc.github.io).

Furthermore, as someone with a degree in Computer Science, that's been programming for nearly two decades I'm not the target audience. We require feedback to understand whether we're hitting the mark, please tell us if you have suggestions for how the website and community could be improved. Either [via GitHub](https://github.com/sig-rpc/sig-rpc.github.io/issues/new/choose) or by contacting SIG-RPC's organising committee at [sig-rpc-managers@society-rse.org](mailto:sig-rpc-managers@society-rse.org).

Soon we'll organise our 3rd open-meeting, in February 2025, to have a live discussion about the website and consider how to move the community forwards. At the end of last year [I was granted an SSI fellowship](https://www.software.ac.uk/news/introducing-2025-fellowship-cohort-insights-and-celebrations) to help me build this community, which provides a small pot of funding that can be used for hosting events. We just need to decide what!

Additionally, this meeting will be considered an EGM as we hope to elect a new member to SIG-RPCs organising committee, following the founding editor changing jobs and being unable to continue in the role.

Robert Chisholm
SIG-RPC Chair
[sig-rpc-managers@society-rse.org](mailto:sig-rpc-managers@society-rse.org)