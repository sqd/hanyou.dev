---
title: "Covering a Boolean Matrix with Minimum Number of Rectangles"
date: 2019-04-08 12:27:30
tags: "Algorithm"
---
The problem (by @rubyleehs):
> Suppose we have a boolean matrix. How do we use the fewest (maybe overlapping) rectangles to cover all the `true` cells and none of the `false` cells?

Given a $n\times n$ matrix, obviously the lowerbound of this problem is $O(n^2)$. So that's what we are aiming for.

Let's first consider a reduced problem. What if instead of an arbitrary matrix, we are given a "mountain" matrix, represented by an array that gives the height of the mountain at each position? We call it a mountain because "caves" are not allowed; or more formally, any cell below a `true` cell in a mountain matrix must be `true`. An example of such a mountain and its representation (`M` = `true` and empty = `false`):

```
 MM  MM
MMMM MMM
MMMMMMMM
MMMMMMMM
34432443 (representation array)
```

One of the minimum covers of 5 rectangles:
```
 33  55
2222 444
11111111
11111111
```

It seems that what we do is pretty simple in this case:

```python
def minimum_cover(mountain):
  while mountain is not empty:
    rectangle = extend the bottom row up, make it as high as possible
    add the rectangle to the answer
    delete the rectangle from the mountain
    if the mountain splits into several smaller ones:
      for sub_mountain in smaller_mountains:
        minimum_cover(sub_mountain)
      return
```

This is very intuitive and in fact, it works for all mountains! We will prove that. But first, we need to think about how we select a rectangle in the third line. By "make it as high as possible", we actually mean "make it as high as the lowest point". So for example in our mountain, first we select the $8\times 2$ rectangle from the bottom (the one marked by 1), because the lowest point in the mountain has height 2. Why does this work?

Let's consider the lowest point $P$ with height 2. We have to cover it by at least one rectangle. So let's make a small $1\times1$ rectangle $R$ covering exactly $P$. But we remember that our rules **allow us to have overlapping rectangles**, so while we are at it, we **might as well** extend $R$ down to the bottom: this allows us to potentially cover more points without influencing the possibility of placing other rectangles. By the same reasoning, we **might as well** extend $R$ to the left and to the right, and because $P$ is the lowest point in the whole range, we can always extend $R$ to the very left and to the very right. To recap:

1. We'll always have to cover $P$ no matter what.
2. We thus add the maximum rectangle $R$ that covers $P$ to our answer. Because
	- It covers $P$.
	- We could potentially cover the most other points, without affecting how we may place other rectangles.

Thus in every step of our algorithm, we are adding a rectangle that is **absolutely neccesary**. And obviously this algorithm terminates. In sum, it produces the optimal answer, i.e. a minimum cover of the mountain.

Now back to the original problem. In the same vein, I propose this algorithm before proving that it is optimal:

```python
for x = 1..n:
  for y = 1..n:
    if M[x, y] == false:
      continue
    make a rectangle R that covers only point (x, y)
    extends R to the right as wide as possible
    extends R above as high as possible
    add R to the answer
    M[R] = false
```

Now obviously, when we reach the 5th line "make a rectangle R...", every cell in $M$ that is (1) not to the right of $(x, y)$ and on the same row, **or** (2) below $(x, y)$ is already false. Graphically:

```
........
?..???..
FFFF*?..
...FFFFF
........
F......F
```

where `*` is the point $(x, y)$ we are considering. Since a region full of `F` is boring, we'll only consider the interesting region, i.e., the cells on the same row with $(x, y)$ or above it.

Consider a rectangle $R=▭AXBCD$ selected by our algorithm, where `A` is $(x, y)$, `B`, `C`, and `D` are the other three corners of the rectangle, and `*` is the uninteresting inner content. You can see that an `F` stops our rectangle from growing even higher, and `X` is the cell below it at the bottom:

```    
FFFFFFFFFF
F    F  FF
   D***C F
F  ***** F
FFFA*X*BFF
```

By way of contradiction, suppose that this is not in an optimal solution. Furthremore, in any optimal solution, there has to be a rectangle $R_A$ that covers `A`. Because $R$ is not in any optimal solution, $R_A$ can't be a subset of $R$, otherwise we **might as well** extend $R_A$ to $R$ and still cover the same points, but this implies that $R$ is in some optimal solution, contradicting with our supposition. Therefore, because obviously $R_A$ cannot be wider than $R$, it must be higher and narrower to not be a subset of $R$:

```    
FFFFFFFFFF
F  **F  FF
   **    F
F  **    F
FFFA*   FF
```

Again, we let $R_A$ as high as possible and "touch the ceiling" because why not? We're not covering fewer points by making it higher, so we **might as well** do that.

I **might as well** stop highlighting "**might as well**" from now on, but do realize that this might-as-well thinking is the center of our algorithm and proof.

Similarly, we need to have a $R_B$ that covers `B`. Similarly, it can't be a subset of $R$ and has to be narrower and higher. We might as well make it as large as possible:

```    
FFFFFFFFFF
F    F**FF
      ** F
F     ** F
FFF   *BFF
```

Because neither $R_A$ nor $R_B$ covers `X`, we also need a $R_X$ that covers `X`. Remember that $R_X$ is in some optimal solution. So as always, $R_X$ can't be a subset of $R$, because otherwise we can extend $R_X$ to $R$, and therefore $R$ is in a optimal solution, contradicting our supposition. But we realize that we cannot make such a $R_X$! It is trapped inside $R$:

```    
FFFFFFFFFF
F    F  FF
   [---] F
F  [   ] F
FFF[ X ]FF
Admiral Ackbar (circa 2019, characterized)
```

Big if true, because this contradicts our supposition that $R$ is not in any optimal solution.

Thus, every $R$ selected by our algorithm is in an optimal solution about the points that are not yet covered. And obviously this algorithm terminates. So we have proved that our algorithm is indeed optimal!

What about time complexity? This analysis is actually a breeze compared to proving its optimality. For the sake of your mouse scroll button I copy the code down here:

```python
for x = 1..n:
  for y = 1..n:
    if M[x, y] == false:
      continue
    make a rectangle R that covers only point (x, y)
    extends R to the right as wide as possible
    extends R above as high as possible
    add R to the answer
    M[R] = false
```

Observe that:
1. Each cell is marked exactly once
2. Each cell is "extended over" by a rectangle exactly once
3. Adding a rectangle to the answer is constant (since we only need its position and size)

Therefore, because there are $n^2$ cells, the whole algorithm runs at $O(n^2)$, theoretical lowerbound!

Homework:
1. What if it is either `C` or `D` that is blocked?
2. Instead of doing row-column ($x$ in the outer loop, $y$ in the inner loop), we can also do column-row. Write the pseudocode and produce a proof for this variation, and determine if it is better.