---
title: "Yet Another Method to Generate Permutations"
date: 2019-08-06 11:58:54
tags: "Algorithm"
---
There are tons of methods to generate every permutation of some elements. And here's another intuitive and efficient one.

Suppose we have $N$ elements $a_1, a_2 ... a_N$ and $answer$ is a 1-index (yeah, burn me) list sized $n!$ (so it can just hold all of the permutations). Suppose that we also have function $f(n)$ which fills $answer[1..n!]$ with the permutation of $a_1, a_2 ... a_n$. Thus if we call $f(1), f(2) ... f(N)$ in sequence, $answer$ should hold what we want.

$f(1)$ is pretty simple. We simply set $answer[1] = [a_1]$.

When we are calling $f(n)$, we know that $f(n-1)$ has been called and $answer[1..(n-1)!] = P$ already holds the permutations of $a_1...a_{n-1}$. We observe that for each permutation $p$ in $P$, if we replace any of $p_i$ with $a_n$ and put $p_i$ in the back instead, we get a new unique (obviously) permutation of $a_1...a_n$:
\\[
    [p_1, p_2 ... p_i ... p_{n-1}] \rightarrow [p_1, p_2 ... a_n ... p_{n-1}, p_i]
\\]

Note that we can also change nothing and just append $a_n$ to the back.

So this is what we do for $f(n)$: we loop through $answer[1..(n-1)!] = P$. For each permutation $p$ in $P$, we generate $n-1$ new permutations by the aforementioned "replacing" and append $a_n$ to the original $p$. Because we're always appending to lists, there is no nasty memory move-around. Because each "replacing" is constant time, we're at the minimal $O(n!)$ complexity.

Here is a Python implementation that passes the [LeetCode problem](https://leetcode.com/problems/permutations/). Because it's real-world, we starts with $n=0$ instead:

```python
def permute(a):
    N = len(a)
    
    # Manual case when n = 0
    answer = [[a[0]]]
    
    # n = [1 .. N)
    for n in range(1, N):
        # i = [0 .. (n-1)!)
        # because len(answer) = (n-1)! anyway
        for i in range(len(answer)):
            # j = [0, n)
            for j in range(n):
                # "replacing" step
                new_perm = list(answer[i]) + [a[n]]
                new_perm[j], new_perm[-1] = new_perm[-1], new_perm[j]
                answer.append(new_perm)
                
            # change nothing and just append a[n]
            answer[i].append(a[n])

    return answer
```