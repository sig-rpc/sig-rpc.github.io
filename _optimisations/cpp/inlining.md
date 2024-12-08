---
level: 1
published: true
authors: Robert Chisholm

name: Inline small functions
language: [C/C++]
subcategory: []
tags: [functions, small]
---

Calling a function has a small overhead, for small (e.g. mathematical) functions this can be high relative to the cost of actually executing the function. By moving small functions into headers and marking them as `inline`, the compiler will normally inline them removing the call overhead when compiler optimisations are enabled. Inlined functions will not be shown by function-level profiling.

<!--more-->

## Example Code

## The Technical Detail
