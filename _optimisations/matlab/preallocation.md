---
level: 1
published: true
authors: Mike Croucher

name: Preallocation
language: [MATLAB]
subcategory: [Core]
tags: [array, loops]
---

When you grow an array dynamically inside a for loop, MATLAB must repeatedly allocate new memory and copy the existing contents. This can significantly degrade performance. Preallocating reserves sufficient contiguous memory upfront, avoiding costly resizing operations later.

<!--more-->

## Example Code

To take advantage of [Preallocation](https://uk.mathworks.com/help/matlab/matlab_prog/preallocating-arrays.html), you should allocate your array outside of the for-loop before using it.

The example below creates an array that grows with every iteration of the loop 

```matlab
tic
x = 0;
N = 10000000;
for k = 2:N
   x(k) = x(k-1) + 5;
end
toc

Elapsed time is 0.380283 seconds.
```

If we allocate the array before entering the loop, the code runs almost 10x faster

```matlab
tic
x = zeros(1,N);
N = 10000000;
for k = 2:N
   x(k) = x(k-1) + 5;
end
toc

Elapsed time is 0.040541 seconds.
```

## The Technical Detail

Prior to MATLAB R2010b, memory allocation worked like this: 

The first time through the loop, the variable x hasn't been assigned to yet, so MATLAB creates it through the assignment to `x(1)`. The second time through the loop assigns to `x(2)`, which doesn't exist yet. So MATLAB allocates enough new memory for two elements and then copies `x(1)` and assigns `x(2)` into the new space. The third time through the loop assigns to `x(3)`, which doesn't exist yet. So MATLAB allocates enough new memory for three elements and then copies `x(1)` and `x(2)` and assigns `x(3)` into the new space.

See the pattern? On the k-th time through the loop, MATLAB has to make a copy of k elements into newly allocated memory. The time required to copy k elements is proportional to k. The time required to execute the entire loop is therefore proportional to the sum of the integers from 1 to n, which is n*(n+1)/2.

Bottom line: memory copying during the assignment statement causes the time required to execute the loop to be proportional to n^2. For large N, this is very slow!

From R2010b onwards, [MATLAB uses a much more efficient memory allocation strategy](https://blogs.mathworks.com/steve/2011/05/16/automatic-array-growth-gets-a-lot-faster-in-r2011a/?s_tid=blogs_rc_2) when it encounters loops like this so you are not 'punished' as much as you used to be if your code follows this pattern. However, it is still sugnificantly less efficient than simply creating the array once and writing to it in the loop.