---
level: 1
published: true
authors: Fred Sonnenwald, Robert Chisholm

name: Vectorisation
language: [MATLAB]
subcategory: [Core]
tags: [array, maths, loops, conditional]
---

MATLAB is optimised to take advantage of vectorisation for mathematical operations between arrays, whereby the processor executes one instruction across multiple variables simultaneously. Vectorisation can perform mathematical operations many times faster than a for loop and even take advantage of conditional logic.

<!--more-->

## Example Code

To take advantage of [vectorisation within MATLAB](https://uk.mathworks.com/help/matlab/matlab_prog/vectorization.html), operations should be performed at an array-level, rather than by looping over the individual elements.

The below example multiplies every element of an array by 2 using a for loop.

```matlab
% Allocate an array of 1 million elements
vals = 1:1000000;
% Multiply all elements by 2
for ii=1:length(vals)
    vals(ii) = vals(ii) * 2;
end
```

To use vectorisation, it can be rewritten as a single line of code

```matlab
% Allocate an array of 1 million elements
vals = 1:1000000;
% Multiply all elements by 2
vals = vals * 2;
```

Not all of your MATLAB code will be this simple though, maybe you have conditional logic which restricts which elements the operation should be performed on.

```matlab
% Allocate an array of 1 million elements
vals = 1:1000000;
% Multiple by 2 all elements divisible by 3, divide other elements by 2
for ii=1:length(vals)
    if mod(ii, 3) == 0
        vals(ii) = vals(ii) * 2;
    else
        vals(ii) = vals(ii) / 2;
    end
end
```

The above example can be written as

```matlab
% Allocate an array of 1 million elements
vals = 1:1000000;
% Produce a boolean array k of whether elements are divisible by 3
k = mod(vals, 3) == 0
% Conditionally divide or multiply elements by 2
vals(k) = vals(k) * 2;
vals(~k) = vals(~k) / 2;
```

Using vectorisation is both more concise and more performant.

## The Technical Detail

Vector instructions, which MATLAB can take advantage of, enable a CPU to apply the same operation to multiple data elements in parallel with a single thread.

Modern CPUs use SIMD (Single Instruction, Multiple Data) instructions to operate on several values packed into a register. Since a typical CPU cache line is 64 bytes, and standard data types like 32-bit or 64-bit floats and integers are 4 or 8 bytes each, 8–16 values can fit within a single cache line and be processed together by a single vector instruction.

However, to take advantage of this, the data must be aligned in memory. Laid out such that it fits neatly into the expected cache line boundaries. MATLAB arrays are explicitly designed for numerical performance, they allocate memory with alignment guarantees, ensuring that vector instructions can be used. Therefore, you just need to make sure where possible you're using operations which support vectorisation.
