---
layout: post
title: Project Euler Problem 53
---

Rather than cherry-pick another PE problem, I'm gonna move right along to the
next one in the sequence.

> **Combinatoric selections**

> How many, not necessarily distinct, values of *n choose r*, for 1 <= n <= 100, are greater than one million?

Naturally, the first thing to do is to define our `choose` function. We can
pull the definition straight from the formula given in the problem definition:

{% highlight haskell %}
choose :: Int -> Int -> Int
choose n r = fact n `div` (fact r * fact (n-r))
    where fact i = product [1..i]
{% endhighlight %}

But this turns out to be a really inefficient way of computing binomial
coefficients (the fancy name for the output of this function). Its inefficient,
since we'll have to calculate three different factorials, and it'll also give
us nonsense answers when the factorials get big enough to cause integer
overflow. Of course, we can avoid overflow by using `Integer` rather than
`Int`, but we'd be better off avoiding this definition altogether.

{% highlight haskell %}
choose :: Int -> Int -> Int
choose _ 0 = 1
choose 0 _ = 0
choose n k = choose (n-1) (k-1) * n `div` k
{% endhighlight %}

This is much better. It uses a recurrence relation for binomial coefficients
that doesn't involve computation of factorials, which I found in the [wikipedia
article for
combinations](http://en.wikipedia.org/wiki/Combination#Number_of_k-combinations).

We still have the problem of overflow, which is unavoidable due to the size of
the values we'll be computing. For instance, `100 choose 50` is over 10^29 --
much higher than the maximum `Int` value. Thankfully, all we need to do is
change our type signature:

{% highlight haskell %}
choose :: Integer -> Integer -> Integer
{% endhighlight %}

The fact that we can change our type signature this easily means that our
`choose` definition is not as type-specific as we're forcing it to be. In fact,
we can make it more abstract:

{% highlight haskell %}
choose :: Integral a => a -> a -> a
{% endhighlight %}

This is possible because our definition uses three functions internally: `(-)`,
`(*)`, and `div`. The first two are defined in the `Num` typeclass, while the
third is defined in the `Integral` typeclass -- which is itself defined in
terms of `Num`. Thus `Integral` is the least specific constraint we have to put
on this function; any `Integral` can work as input.

Now, telling GHC to use a specific `Integral` instance is as simply as
providing a type annotation. Doing so in GHCI shows us the difference between
the overflowing `Int` and the infinite-precision `Integer`:

    > 100 `choose` 48 :: Integer
    93206558875049876949581681100
    > 100 `choose` 48 :: Int
    -9982283485397623

We need to make a list of all the values of `n choose k` for each `n` between 1
and 100 inclusive, and for each `k` between 1 and `n` inclusive. We can define a
function that will give us all values for `k` in terms of some input `n`:

{% highlight haskell %}
choices :: Integral a => a -> [a]
choices n = map (choose n) [1..n]
{% endhighlight %}

...and then map that function over all the values of `n`. This will give us a
list of lists, but for our purposes it would be best to have a flat list. Thus,
we can use `concatMap`.

{% highlight haskell %}
values :: Integral a => [a]
values = concatMap choices [1..100]
{% endhighlight %}

We're not defining a function here, just a bare list of `Integrals`. We can
filter the list by the predicate given in the problem, and find the length of
the filtered list.

{% highlight haskell %}
solution :: Int
solution = length $ filter (>1000000) values
{% endhighlight %}

This is all well and good, but GHC will give us a warning if we compile this
definition. It warns that it's defaulting to `Integer` as the instance for
the values in `values`. That's fine for our purposes, since we wanted the
precision of `Integer`s, but I hate having to deal with the warning (especially
since it shows up in `ghc-mod`, my linter.)

To avoid it, we can inline a type annotation for our use of values.

{% highlight haskell %}
solution = length $ filter (>1000000) (values :: [Integer])
{% endhighlight %}

However, its a little cleaner, and a lot more sensible, to put this annotation
in the type signature for `values` itself. It doesn't make that much sense to
have `values` be polymorphic, especially since we know that most instances of
`Integral` are going to give us overflowing, nonsense results.

Lastly, we can note that `concatMap` is the definition for `>>=` in the list
monad. So if we want to look a bit fancier, we can rewrite values as

{% highlight haskell %}
values = [1..100] >>= choices
{% endhighlight %}

Using `>>=` makes it a little less obvious what's going on, though experienced
Haskellers will be able to tell that we're feeding all of the members of the
list through `choices`, one after the other, and flattening the result.
However, I prefer the more explicit version, where we use `concatMap` directly.
