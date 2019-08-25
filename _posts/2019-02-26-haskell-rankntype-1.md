---
title: "What does Haskell RankNTypes do? (1) - Semantics"
date: 2019-02-26 22:33:26
tags: "Haskell"
---
Consider this function:
```haskell
tuple2Int :: (x -> Int) -> (a, b) -> (Int, Int)
tuple2Int f (a, b) = (f a, f b)
```
...which wouldn't work. Why? Because of `f a`, the type checker deduces that `x~a` (`x` is equivalent to `a`); because of `f b`, the type checker also deduces that `x~b`. Therefore, `a~x~b`, which is obviously false for most cases.

With `RankNTypes`, we can instead write the type signature of the function as `tuple2Int' :: (forall x. x -> Int) -> (a, b) -> (Int, Int)`. This way, the compiler knows that it should use two seperate `x`'s for `a` and `b`. It therefore deduces that `x1~a` and `x2~b` and because it is not enforced that `x1~x2`, there is no problem at all!

### Go Deeper
What's going on?

First, we need to learn the cruel and ugly fact that NO so-called "generic" function in Haskell is actually generic in essence. What does this mean? Let's consider `length :: [a] -> Int`. It looks pretty generic to me as I can use it on any type of list, you may say. This is true, but what happens under the hood is that the compiler _specializes_ a version of the function with concrete type (i.e. no type variable like `a`) everytime you use it. For example, when you do `length [1, 2, 3]`, a specialized version `length(Int) :: [Int] -> Int` is magically created by the compiler to accomodate you; when you do `length "abcdefg"`, similarly a `length(Char) [Char] -> Int` is created. Just remember this for now.

Just as we use `\x -> ...` to define a lambda function with variable `x`, we use `forall a. ...` to define a type with type variable `a`. Therefore, suppose that we can construct the **type of a function** by a data constructor `FunctionType a b`, where `a` and `b` are **values** that represent types, our `(forall x. x -> Int)` in the last paragraph would be equivalent to `\x -> FunctionType x Int`. You also need to remember this.

And then, a crash course in lambda calculus. Consider this lambda function: `\x y -> x + y + z`. We call `x` and `y` **bound variables**, because their appearance is bound by the `\x y` part of the function. We call `z` a **free variable**, because it is defined "elsewhere" and is thus not bound by the `\x y` part. End of the crash course.

With the three things we just learned, we can touch on the problem. Firstly, we have a rule that **no type variables in a type contruction can be free**. For example, we can't write a type-level function like `\x -> FunctionType x y`, or formally `forall x. x -> y`, because `y` is free. This is a very strict rule and obviously we don't want to be scolded by the compiler every now and then just because of it, so we have another catch-all rule: **if a type variable is not explicitly bound, it is then treated as bound at the _beginning_ of the expression.** What does this mean? Suppose we have a type `a -> a` (or `FunctionType a a` with our syntax). Because you see no `forall` in its type signature, technically `a` is free. This is when the rule kicks in: because `a` is not _explicitly_ bound, we treat the whole type as `forall a. a -> a` (or `\a -> FunctionType a a`) and voila! The first rule is automatically satisfied. How nice.

Generic functions in Haskell are all defined this way. `id`, for example, has type `forall a. a -> a`. Our `tuple2Int`, because it has `x`, `a` and `b` that all appear to be free, has type `forall x a b. (x -> Int) -> (a, b) -> (Int, Int)` by the second rule. This is where our problem arises: when we are using this function, the compiler needs to create a concrete version of it (in other words, it needs to assign types to `x`, `a` and `b`). Furthremore, because `x` is scoped on the **whole** type expression, it must takes the same value everywhere. An example:
```haskell
foo = (\x -> x + (\y -> x + 3) 4) 5
bar = (\f -> f 2 + f 3 + f 4) (\x -> x + 2)
```
In `foo`, because `x` is bound at the very beginning, it is scoped over the whole expression. Therefore, it must always takes the value `5`. On the contrary, in `bar`, because `x` is bound only inside the second pair of brackets, it can take different values of `2`, `3`, and `4` elsewhere (i.e. inside the first pair of brackets).

Back to `tuple2Int`. What type should the compiler assign to `x`? It sees that `a` can be applied to `x -> Int`, so `a` and `x` must be the same. Similar deduction goes for `b` and `x`. Therefore, unless `a` and `b` are the same type, we would have a type contradiction.

Now make sure that you understand the problem, and we'll proceed to its solution.