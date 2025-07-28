---
level: 2
published: true
authors: Fred Sonnenwald, Robert Chisholm

name: GPU Arrays
language: [MATLAB]
subcategory: [Parallel Computing Toolbox]
tags: [array, gpu, cuda]
---

MATLAB makes it easy to run matrix operations on an [NVIDIA GPU](https://uk.mathworks.com/help/parallel-computing/gpu-computing-requirements.html). There is a cost for moving data to and from the GPU, but if your entire algorithm can be done in matrix or vector operations, and your data is big enough, it can be faster.

<!--more-->

MATLAB uses the `gpuArray` type to represent arrays used in GPU computation, and it [supports most core MATLAB matrix and vector operations](https://uk.mathworks.com/help/referencelist.html?type=function&capability=gpuarrays).

## Example Code

For example if we wish to perform some operations on a large matrix.

```matlab
% Allocate a 4000x4000 Matrix
l = 4000; % Size of Matrix
% Some random matrices to operate on
A = rand(l,l);
B = rand(l,l);
C = rand(l,1);

% Time some CPU operations
tic
D = A .* B;
E = D * C;
toc
```

This takes around 19 ms to compute.

In contrast, performing the same operation on the GPU and copying the data back only takes 2 ms. Note, it may run slower the first time the code is run while MATLAB compiles an internal CUDA kernel.


```matlab
% Allocate a 4000x4000 Matrix
l = 4000; % Size of Matrix
% Some random matrices to operate on
A = rand(l,l);
B = rand(l,l);
C = rand(l,1);

% Copy the matrices to thee GPU
A_gpu = gpuArray(A);
B_gpu = gpuArray(B);
C_gpu = gpuArray(C);

% Time some GPU operations
tic
D_gpu = A_gpu .* B_gpu;
E_gpu = D_gpu * C_gpu;
% Copy the data back from the GPU
E_gpu = gather(E_gpu);
toc
```

<!-- This could be fleshed out further, timing compute and memcpy separate. -->

## The Technical Detail

GPU parallelism requires distinct accelerator hardware suitable for some highly parallel tasks. In contrast to a CPU’s up to 100 parallel cores, GPUs require 3 order of magnitude more threads to be fully saturated (e.g. 100,000 threads).

To help enhance GPU parallelism, MATLAB provides support functions such as `pagefun` to help [parallelise small GPU operations](https://uk.mathworks.com/help/parallel-computing/improve-performance-of-small-matrix-problems-on-the-gpu-using-pagefun.html).

Our parallel section of the website has a [GPU parallelism explainer](/parallel/#gpu).
