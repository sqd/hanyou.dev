---
title: "Be Careful of Unsafe-Indexing and Modifying C++ Containers Simultaneously"
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