---
level: 1
published: true
authors: Robert Chisholm

name: gprof
language: [C/C++, Fortran]
style: [Function-Level]
website: https://ftp.gnu.org/old-gnu/Manuals/gprof-2.9.1/html_mono/gprof.html
---

`gprof` is the GNU profiler, suitable for function-level profiling code compiled with C, C++ & Fortran compiled with the GNU, PGI, Intel & other compilers. If already compiling your code with a supported compiler, it's relatively easy to setup and use. As a command-line profiler output is provided in a plaintext format, this can be challenging to interpret for more nuanced and subtle bottlenecks.

<!--more-->

## QuickStart

**1.** In order to profile with `gprof`, you should pass the flag `-pg` to a Release (optimised) build at compile time. *If using an Intel compiler `-pg` has been deprecated, and `-p` should be used instead.*

e.g. 

```sh
# Compile hello_world.c with -pg -O3
gcc -pg -O3 -o hello_world hello_world.c
# or
# Compile hello_world.f with -pg
f77 -o hello_world -fast -pg hello_world.f
# or
# Configure the cmake project to build with -pg
# Some of these arguments will be redundant, so you may receive a warning that they're unused
cmake -DCMAKE_C_FLAGS=-pg -DCMAKE_CXX_FLAGS=-pg -DCMAKE_Fortran_FLAGS=-pg -DCMAKE_EXE_LINKER_FLAGS=-pg -DCMAKE_SHARED_LINKER_FLAGS=-pg ..
```

**2.** Now you can execute the program compiled with `-pg` like normal, this will output a profiling dump of the run to `gmon.out`. The program must exit successfully.

```sh
./hello_world
```

**3.** Finally, you should pass both `gmon.out` and the program to `gprof`, to produce the human-readable output.

```sh
# > analysis.txt, pipes the output of the command to file
gprof hello_world gmon.out > analysis.txt
```

## Interpreting Output

The output file `analysis.txt` can be opened with a text editor, it contains several sections.

### Flat Profile

This first section provides a table that identifies functions in order of the proportion of execution time they occupied.

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

After the table, a key to the columns is provided.

<details markdown="block">
<summary>Flat Profile Key</summary>

```
 %         the percentage of the total running time of the
time       program used by this function.

cumulative a running sum of the number of seconds accounted
 seconds   for by this function and those listed above it.

 self      the number of seconds accounted for by this
seconds    function alone.  This is the major sort for this
           listing.

calls      the number of times this function was invoked, if
           this function is profiled, else blank.

 self      the average number of milliseconds spent in this
ms/call    function per call, if this function is profiled,
	       else blank.

 total     the average number of milliseconds spent in this
ms/call    function and its descendents per call, if this
	       function is profiled, else blank.

name       the name of the function.  This is the minor sort
           for this listing. The index shows the location of
	       the function in the gprof listing. If the index is
	       in parenthesis it shows where it would appear in
	       the gprof listing if it were to be printed.
```

</details>

### Call Graph

Following the flat profile, is the call graph. This further breaks down the information from the flat profile into the unique calling hierarchies. This is much longer and can be harder to interpret, however it may provide useful context if a function highlighted by the flat profile is called from many locations.


```
...
-----------------------------------------------
                0.00    0.00       1/4000003     LB::latticeBolzmannInit(std::vector<cylinder, std::allocator<cylinder> >&, std::vector<wall, std::allocator<wall> >&, std::vector<particle, std::allocator<particle> >&, std::vector<object, std::allocator<object> >&, bool, bool) [20]
                0.01    1.39 2000001/4000003     LB::latticeBolzmannStep(std::vector<elmt, std::allocator<elmt> >&, std::vector<particle, std::allocator<particle> >&, std::vector<wall, std::allocator<wall> >&, std::vector<object, std::allocator<object> >&) [5]
                0.01    1.39 2000001/4000003     LB::latticeBoltzmannFreeSurfaceStep() [6]
[4]     55.9    0.02    2.78 4000003         LB::cleanLists() [4]
                0.97    1.81 12000009/16000013     void std::__heap_select<std::reverse_iterator<__gnu_cxx::__normal_iterator<double*, std::vector<double, std::allocator<double> > > >, __gnu_cxx::__ops::_Iter_less_iter>(std::reverse_iterator<__gnu_cxx::__normal_iterator<double*, std::vector<double, std::allocator<double> > > >, std::reverse_iterator<__gnu_cxx::__normal_iterator<double*, std::vector<double, std::allocator<double> > > >, std::reverse_iterator<__gnu_cxx::__normal_iterator<double*, std::vector<double, std::allocator<double> > > >, __gnu_cxx::__ops::_Iter_less_iter) [3]
-----------------------------------------------
                0.00    2.18 2000001/2000001     goCycle(IO&, DEM&, LB&) [2]
[5]     43.6    0.00    2.18 2000001         LB::latticeBolzmannStep(std::vector<elmt, std::allocator<elmt> >&, std::vector<particle, std::allocator<particle> >&, std::vector<wall, std::allocator<wall> >&, std::vector<object, std::allocator<object> >&) [5]
                0.01    1.39 2000001/4000003     LB::cleanLists() [4]
                0.16    0.30 2000001/16000013     void std::__heap_select<std::reverse_iterator<__gnu_cxx::__normal_iterator<double*, std::vector<double, std::allocator<double> > > >, __gnu_cxx::__ops::_Iter_less_iter>(std::reverse_iterator<__gnu_cxx::__normal_iterator<double*, std::vector<double, std::allocator<double> > > >, std::reverse_iterator<__gnu_cxx::__normal_iterator<double*, std::vector<double, std::allocator<double> > > >, std::reverse_iterator<__gnu_cxx::__normal_iterator<double*, std::vector<double, std::allocator<double> > > >, __gnu_cxx::__ops::_Iter_less_iter) [3]
                0.32    0.00 2000001/2000001     LB::streaming(std::vector<wall, std::allocator<wall> >&, std::vector<object, std::allocator<object> >&) [11]
-----------------------------------------------
...
```

Again it is followed by an explanation for understanding the output.

<details markdown="block">
<summary>Call Graph Help Text</summary>
```
This table describes the call tree of the program, and was sorted by
 the total amount of time spent in each function and its children.

 Each entry in this table consists of several lines.  The line with the
 index number at the left hand margin lists the current function.
 The lines above it list the functions that called this function,
 and the lines below it list the functions this one called.
 This line lists:
     index	A unique number given to each element of the table.
		Index numbers are sorted numerically.
		The index number is printed next to every function name so
		it is easier to look up where the function is in the table.

     % time	This is the percentage of the `total' time that was spent
		in this function and its children.  Note that due to
		different viewpoints, functions excluded by options, etc,
		these numbers will NOT add up to 100%.

     self	This is the total amount of time spent in this function.

     children	This is the total amount of time propagated into this
		function by its children.

     called	This is the number of times the function was called.
		If the function called itself recursively, the number
		only includes non-recursive calls, and is followed by
		a `+' and the number of recursive calls.

     name	The name of the current function.  The index number is
		printed after it.  If the function is a member of a
		cycle, the cycle number is printed between the
		function's name and the index number.


 For the function's parents, the fields have the following meanings:

     self	This is the amount of time that was propagated directly
		from the function into this parent.

     children	This is the amount of time that was propagated from
		the function's children into this parent.

     called	This is the number of times this parent called the
		function `/' the total number of times the function
		was called.  Recursive calls to the function are not
		included in the number after the `/'.

     name	This is the name of the parent.  The parent's index
		number is printed after it.  If the parent is a
		member of a cycle, the cycle number is printed between
		the name and the index number.

 If the parents of the function cannot be determined, the word
 `<spontaneous>' is printed in the `name' field, and all the other
 fields are blank.

 For the function's children, the fields have the following meanings:

     self	This is the amount of time that was propagated directly
		from the child into the function.

     children	This is the amount of time that was propagated from the
		child's children to the function.

     called	This is the number of times the function called
		this child `/' the total number of times the child
		was called.  Recursive calls by the child are not
		listed in the number after the `/'.

     name	This is the name of the child.  The child's index
		number is printed after it.  If the child is a
		member of a cycle, the cycle number is printed
		between the name and the index number.

 If there are any cycles (circles) in the call graph, there is an
 entry for the cycle-as-a-whole.  This entry shows who called the
 cycle (as parents) and the members of the cycle (as children.)
 The `+' recursive calls entry shows the number of function calls that
 were internal to the cycle, and the calls entry for each member shows,
 for that member, how many times it was called from other members of
 the cycle.
```

</details>

## Limitations

Whilst `gprof` will profile code that includes OpenMP, it can produce spurious timing information if OpenMP execution is present. As such you may also want to disable compiling with OpenMP, and profile the serial execution of your code instead. The process for disabling OpenMP at compile time will differ between projects, so check the documentation.
<!-- More broadly, does it work with parallel C/C++? -->
