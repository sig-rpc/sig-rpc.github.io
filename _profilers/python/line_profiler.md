---
level: 1
published: true
authors: Robert Chisholm

name: line_profiler
language: [Python]
style: [Line-Level]
website: https://kernprof.readthedocs.io/en/latest/
---

`line_profiler` is a Python package for line-level profiling. It is a command-line profiler, that can also be used within Jupyter Notebooks. Functions marked with the `@profile` decorator are profiled when `line_profiler` is enabled during execution.

<!--more-->

## QuickStart

**1.** `line_profiler` can be installed via `pip` or `conda`.

```sh
# install via pip (may not work on MacOS)
pip install line_profiler[all]
# or
# install via conda
conda install conda-forge::line_profiler
```
**2.** Functions to be profiler should be marked with the `@profile` decorator.

```py
# Import the @profile decorator
from line_profiler import profile 

# Attach the decorator to the functions to be profiled
@profile
def is_prime(number):
    if number < 2:
        return False
    for i in range(2, int(number**0.5) + 1):
        if number % i == 0:
            return False
    return True
    
print(is_prime(1087))
```

**3.** Run the Python code via `line_profiler`.

```py
python -m kernprof -lvr my_script.py
```

## Interpreting Output

At exit, a table will be printed to the command-line for the profiling results of each targeted function.

<details markdown="block">
<summary>Example Output</summary>

*If running this in a terminal, you should see rich output with highlight colours.*

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

The columns have the following definitions:

| Column | Definition |
|---------|---------------------------------------------------|
| `Line #`  | The line number of the relevant line within the file (specified above the table). |
| `Hits` | The total number of times the line was executed. |
| `Time` | The total time spent executing that line, including child function calls. |
| `Per Hit` | The average time per call, including child function calls (`Time`/`Hits`). |
| `% Time` | The time spent executing the line, including child function calls, relative to the other lines of the function. |
| `Line Contents` | A copy of the line from the file. |

</details>

## Jupyter Notebooks

If you're more familiar with writing Python inside Jupyter notebooks you can use `line_profiler` directly from inside notebooks. However it is still necessary for the code you wish to profile to be placed within a function.

First `line_profiler` must be installed and it's extension loaded.

```py
!pip install line_profiler
%load_ext line_profiler
```

Following this, you call `line_profiler` with `%lprun`.

```py
%lprun -f profiled_function_name entry_function_call()
```

The functions to be line profiled are specified with `-f <function name>`, this is repeated for each individual function that you would otherwise apply the `@profile` decorator to.

This is followed by calling the function which runs the full code to be profiled.

For the above is_prime example it would be:

```py
%lprun -f is_prime is_prime(1087)
```

This will then create an output cell with any output from the profiled code, followed by the standard output from `line_profiler`. *It is not currently possible to get the rich/coloured output from `line_profiler` within notebooks.*

## Limitations

`line_profiler` does not support line profiling a whole Python application, if your Python code to be profiled exists in the root of a Python file, you will need to move it inside a function prior to profiling.

As a deterministic profiler, rather than a sampling profiler, it captures data about every executed line. Therefore, it can lead to the profiling execution being significantly slower if profiling large functions due to the overhead of data collection.
