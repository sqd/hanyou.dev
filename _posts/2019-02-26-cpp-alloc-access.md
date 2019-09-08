---
title: "Be Careful of Direct-Indexing and Modifying C++ Containers Simultaneously"
date: 2019-02-26 21:19:27
tags: "C++"
---
There was this weird C++ bug I encountered, minimally reproducible by
```cpp
#include <iostream>
#include <vector>

std::vector<int> vec;

int alloc(){
    vec.push_back(0);
    return vec.size();
}

int main(){
    vec.push_back(0);
    vec[0] = alloc();
    std::cout << vec[0] << std::endl;
}
```

After the first line of `main`, `vec` has size 1. In the second line of `main`, after the first line of `alloc`, `vec` has size 2. We therefore expect that `vec[0] == 2`. **This is not the case.**

In the first line of `alloc`, `vec` grows from size 1 to 2. My GCC's STL implementation thus realloc a space of size 2 and moves the existing content there. This reallocation invalidates the original memory space, at which `vec[0]` is pointing to. So sometimes you would get a `SEGFAULT` out of this.

**Solution:** `vec.assign(0, alloc());`

So obviously the root cause of this problem is undefined behavior in the execution order of indexing and the evaluation of the value (the bug emerges if indexing happens first). In the solution above, per standard, the parameters are evaluated before the `vec.assign` call so we are fine.

**The compiler is theoretically unable to detect this perfectly**. Obviously the compiler can issue warnings, but it cannot perfectly detect only and every case of such undefined behaviors. Otherwise we could just put any program in the function call and solve the Turing Halting Problem.

If you understand what's going on completely, ignore the following part.

So at the first glance, I was thinking: "hmmm this makes sense if we're using and moving around a raw array, but isn't the `[]` operator of a vector just disguised function call?".

Indeed it is. Suppose that `T& operator[](size_t i) {...}` is renamed to `T& bracket(size_t i) {...}` for clarity's sake, then `vec[0] = alloc()` is translated into:

```cpp
int* ref = &vec.bracket(0);
int val = alloc();
*ref = val;
```

Now everything is defined, but obviously after `alloc`, `ref` becomes a dangling pointer.

Side notes, barring Turing Halting Problem, if a compiler can detect this, we'd have no use of rust and such.