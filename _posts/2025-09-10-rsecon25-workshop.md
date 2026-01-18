---
layout: post
title: RSECon25 Workshop
date: 2025-09-10 15:30:00
author: Robert Chisholm (Chair)
categories: [event, news]
---

On the 10th September we're hosting our first workshop at [RSECon25](https://rsecon25.society-rse.org/) (15:30-17:00 in OC0.04).


[This workshop](https://virtual.oxfordabstracts.com/event/75166/submission/12) will invite attendees to contribute to our knowledge-base, whether that's contributing to [optimisation](/optimisations) or [profiler](/profilers) mini-guides, or reviewing the existing ones.

## Pre-requisites

This workshop is appropriate for attendees that know any programming language used by researchers (e.g. Python, R, C/C++, etc), regardless of their expertise. The exercises available will suit attendees from novice (e.g. reviewing the accessibility of existing materials) to expert (e.g. proposing or drafting new guidance) .

## Preparation

This workshop doesn't require any advanced tooling, everything can be completed with your favourite web browser and text editor (and a GitHub account). We'll bring along some paper forms too, for anyone who fancies a break from their laptop.

You may however want to reflect on what you can contribute ahead of the workshop. Take a look at the [existing optimisations](/optimisations), are these applicable to the language you use? Or are there similar performance patterns you've come across in your own work?

If you have some code you could profile during the workshop, bring that along, you can contribute a new, or review an existing [profiler quick-start guide](/profilers) instead, and maybe profiling your own code will highlight a new performance pattern.

Don't worry if you haven't got time to prepare ahead of the workshop, we'll be bringing along a small backlog of known optimisations that we're yet to document, or you can help test and review existing content.

## Guidance

You can find the intro slides to the workshop on [Google Slides](https://docs.google.com/presentation/d/15Lult4wHnJnE5dyoJD7uksLhtfPoyOzylHS5koJXF1Y/edit?usp=sharing).

Please open the [collaborative document](https://semestriel.framapad.org/p/sig-rpc-ag09) to use for the event, this contains:

- A list of links to existing articles to review/edit
- A list of new ideas which haven't yet been drafted, claim one or add your own by attaching your name

### Task 1 - Reviewing

Open either the [Profiling](https://sig-rpc.github.io/profilers/) or [Optimisations](https://sig-rpc.github.io/optimisations/) knowledge-base, and select a language you know and pick a guide.

Review some of the existing content, and provide any feedback in an [issue on GitHub](https://github.com/sig-rpc/sig-rpc.github.io/issues/new/choose) (one per guide please!).


### Task 2 - Editing

[Fork the git repository](https://github.com/sig-rpc/sig-rpc.github.io/fork) and make a clone.

You can find existing guides in the respective `_profilers` and `_optimisations` directories.
Find guides that you're familiar with, and submit a [pull request](https://github.com/sig-rpc/sig-rpc.github.io/pulls?q=sort%3Aupdated-desc+is%3Apr+is%3Aopen) with your changes.

We endeavour to merge input from as many contributors as possible, but remember the target audience are researchers without significant formal programming training!

### Task 3 - Creation

[Fork the git repository](https://github.com/sig-rpc/sig-rpc.github.io/fork) and make a clone.

Locate or add your suitable idea to the [collaborative document](https://semestriel.framapad.org/p/sig-rpc-ag09), this will avoid multiple people submitting independent drafts of the same idea.

Create a new markdown file in the corresponding `_profilers/<language>` or `_optimisations/<language>` directory and submit a [pull request](https://github.com/sig-rpc/sig-rpc.github.io/pulls?q=sort%3Aupdated-desc+is%3Apr+is%3Aopen).

Use an existing guide (e.g. [profiler](https://github.com/sig-rpc/sig-rpc.github.io/blob/master/_profilers/python/cprofile.md?plain=1), [optimisation](https://github.com/sig-rpc/sig-rpc.github.io/blob/master/_optimisations/python/list-comprehension.md)) to copy the structure of the YAML header, and remember to include `<!-- more -->` to denote where the body content shown in the index ends.

*Even if you don't finish during this session, please submit a draft PR with what you've started. We can then either remind you to finish, or finalise it ourselves!*

### Paper Tasks

We've tried our best to make it possible for a few people to complete the workshop on paper, so you will find some sheets/pens on each table.

-----------

Feel free to chat with the presenters during the activity to discuss ideas, or ask questions!