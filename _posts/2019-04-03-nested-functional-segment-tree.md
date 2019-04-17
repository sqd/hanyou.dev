---
title: Nested Chairman Tree (Functional Segment Tree), and Beyond
date: 2019-04-03 22:27:06
tags: ["Algorithm", "Data Structure"]
---

Recently I ran into one (of the millions of variations of) sequence maintainence problem (BZOJ3065). We need to support insertion/deletion, and querying the $k$th smallest element in a given interval. Hzwer seems to have come up with a [solution](http://hzwer.com/4572.html) that involves a scapegoat-tree-nested segment tree, which I can't say confidently that I understand. So I come up with this solution of my own, with comparable complexity.

Let's start with refreshing our memory on the most classical version of nested segment tree, the one that we use to maintain a sequence that supports modification and querying the $k$th smallest element in a given interval. Firstly, recall how we would approach this problem if we don't need to support modification: if we care about time more than memory, we use a <span style="background: black; color: black;">partition tree</span>; if we care about memory more (and don't mind debugging), we use a <span style="background: black; color: black;">chairman tree</span>.

|                | Query         | Build | Space     |
| -------------  |:-------------:| :----:| :-------: |
| Partition Tree | $\Theta(\log n)$ | $\Theta(n\log n)$ | $\Theta(n\log n)$ |
| Chairman Tree  | $O(\log^2 n)$ | $O(n\log n)$ | $O(n\log n)$ |

Now we use a similar approach to support modification. In essence, both partition tree and chairman tree run a query by looking up how many elements enter the left son in the given interval. This is easy to implement with an array without modification, but with modification, we need some tricks. Let's first formalize the task:

> Suppose on a node, we have a sequence $s_1, s_2 ... s_n$, and an associated sequence $b_1, b_2 ... b_n$, where $b_i = 1$ iff $s_i$ enters the left son, otherwise $b_i = 0$. Thus for a partition-tree-like data structure to work, we need to efficiently run the query $sum(b_x, b_{x+1} ... b_y)$ for any interval $[x, y]$, and also efficiently flipping one of the $b$'s. How should we do this?

This sounds very familiar! We know that there are at least two data structures - <span style="background: black; color: black;">binary indexed tree (BIT) and segment tree</span> - that supports fast query of interval sum and fast modification of single elements. So our approach: contruct a partition tree/chairman tree as usual, but instead of putting an array in each node, we put in a BIT or a segment tree. How is the complexity? We still need to go through $\log n$ internal nodes to find an leaf. For a depth-$k$ internal node, we have $\frac{n}{2^k}$ elements on it; throwing in the logarithm for BIT/segment tree, we spend $O(\log \frac{n}{2^k})$ time on this node. So summing the time we spend on the whole path:
\\[
\log n + \log(n/2) + \log(n/4) ...
\\\\= \log(n/2 \times n/4 \times ...)
\\\\(\text{when }n=2^{odd})\approx \log(\frac{n^{\log n}}{2^{\frac{\log^2 n}{2}}})
\\\\= \log^2n - \frac{\log^2n}{2}\log2 = O(\log^2n)
\\]


Having refreshed our memories on this reduced problem, let's tackle on our original enemy: sequence maintainence with insertion/deletion, and querying the kth smallest element in a given interval. Now BIT/segment tree won't do, because they do not support insertion/deletion. But there is one data structure that does, which is a <span style="background: black; color: black;">Splay Tree</span>!

Oof so many trees. Anyway, if we plug in a splay tree into every node of a chairman tree, we get a data structure that perfectly solves our task. (I mean, $O(\log^2 n)$ is not bad at all in practice). A trick: if the insertion/deletion/modification operations are random, don't even bother to do the rebalance splay step, which would multiply the constant factor by 3. However, if they are *ordered*, splay could degenerate into a linked list quickly! So if you have some operations that you know are ordered, it may be a good idea to shuffle them (if that makes sense in that specific scenario) before putting them into this nested monstrosity.

Look at how we went from array to segment tree/BIT and then to splay! There seems to be a pattern here: if there is a type of operations that can be done on a data structure $D$ in $T(n)$ time, we can plug $D$ in a chairman tree, and get a $\sum_i^{\log n}T(k)\log k, k=\frac{n}{2^i}$ complexity when performing that operation. Now let's do some homework:
1. What if in addition, I want to support negating all numbers in an interval?
2. What if in addition, I want to support setting all numbers below a threshold $s$ to zero in an interval?