---
title: Nested Functional Segment Tree, and Beyond
date: 2019-04-03 22:27:06
tags: ["Algorithm", "Data Structure"]
---

Recently I ran into one (of the millions of variations of) sequence maintainence problem (BZOJ3065). We need to support insertion/deletion, and querying the kth smallest element in a given interval. Hzwer seems to have come up with a [solution](http://hzwer.com/4572.html) that involves a scapegoat-tree-nested segment tree, which I can't say confidently that I understand. So I come up with this solution of my own, with comparable complexity.

Let's start with refreshing our memory on the most classical version of nested segment tree, the one that we use to maintain a sequence that supports modification and querying the kth smallest element in a given interval. Firstly, recall how we would approach this problem if we don't need to support modification: if we care about time more then memory, we use a partition tree; if we care about memory more (and don't mind debugging), we use a [chairman tree](https://cs.stackexchange.com/questions/65312/what-is-the-chairman-tree).

> Partition Tree: Query Θ(logn), Build Θ(nlogn), Memory Θ(nlogn)
>
> Chairman Tree: Query O(logn\*logn), Build O(nlogn), Memory O(nlogn)

Even though now we need to support modification, we can use a similar approach. In essence, both partition tree and chairman tree run a query by looking up how many elements enter the left son in the given interval. This proves to be easy without modification, but with modification, we need to do some tricks. Let's first formalize the task:

> Suppose we have a sequence <img style="display: inline;" class="nofancybox" src="https://quicklatex.com/cache3/79/ql_f3cbd8fb8cec3c54e4cab899bfd74f79_l3.png">, and an associated sequence b_1, b_2 ... b_n, where b_i = 1 iff s_i enters the left son, otherwise b_i = 0. Thus for a partition-tree-like data structure to work, we need to efficiently run the query sum(b_x, b_(x+1) ... b_y) for any interval [x, y], and also efficiently flipping one of the b's. How should we do this?

This sounds very familiar! We know that there are at least two data structures - binary indexed tree (BIT) and segment tree - that supports fast query of interval sum and fast modification of single elements. So our approach: contruct a partition tree/chairman tree as usual, but instead of putting an array in each node, we put in a BIT or a segment tree. How is the complexity? We still need to run down logn nodes to find an exact element, so on the nodes we spend logn + log(n/2) + log(n/4) ... = log(n/2 \* n/4 \* ...) ~= log(n^logn / 2^ (logn/2)\*logn) = lognlogn - 1/2 lognlognlog2 = O(lognlogn).

Having refreshed our memories on this reduced problem, let's tackle on our original enemy: sequence maintainence with insertion/deletion, and querying the kth smallest element in a given interval.

(TODO: Finish this later)
