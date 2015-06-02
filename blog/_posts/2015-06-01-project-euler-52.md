---
layout: post
title: Project Euler Problem 52
---

In the interest of getting in the habit of blogging, I'm committing myself to
writing a post a day. This is probably a totally unrealistic goal, but I'm
primarily trying it to see how well I can keep up. If I don't have anything
more important to post about, I'll tackle a Project Euler problem in whatever
language I feel like using at the time (probably Python or Haskell). This one
is my first!

> **Permuted Multiples**

> It can be seen that the number, 125874, and its double, 251748, contain
> exactly the same digits, but in a different order.

> Find the smallest positive integer, x, such that 2x, 3x, 4x, 5x, and 6x,
> contain the same digits.

This is a pretty early, pretty simple PE problem. I prefer a top-down approach.

{% highlight haskell %}
module Euler52 where

solution :: Int
solution = firstOf matching integers
    where integers = [1..]
{% endhighlight %}

Ultimately, our solution is going to be an `Int`. We can get a nice,
declarative definition of the solution by introducing a couple identifiers:
`firstOf` and `matching`. `integers` is locally scoped using a `where` clause
and gets defined as an infinite list (the Peano numbers).

Here's `firstOf`:

{% highlight haskell %}
firstOf :: (a -> Bool) -> [a] -> a
firstOf f = head . filter f
{% endhighlight %}

It takes a predicate and a list of inputs to the predicate, testing each one to
see if it satisfies, and returning the first one that does. Because Haskell is
lazy, it won't bother filtering the rest of the list once it finds the first
result.

Now we have the scaffolding to find the smallest positive integer that
satisfies some predicate. Now, we need to define our predicate (`matching`).

{% highlight haskell %}
matching :: (Int -> Bool)
matching n = allPerms $ map (*n) [2..6]
{% endhighlight %}

First we make a list of all the multiples we need (2x, 3x, 4x, 5x, and 6x). We
do this by mapping the function `(*n)` over a list of the multipliers. Then, we
use another predicate, `allPerms`, to see if this all the numbers in this list
are digit-wise permutations of each other.

{% highlight haskell %}
allPerms :: [Int] -> Bool
allPerms = allEq . map (sort . show)
    where allEq xs = and $ zipWith (==) xs (tail xs)
{% endhighlight %}

This is where most the actual work is done in this solution -- everything else
is really just scaffolding and boilerplate. First we `show` each element in the
list, turning the `Int`s into `String`s. Then we `sort` the strings, which
requires us to add an import of `Data.List` below our module declaration.

The only thing left is to see if all the sorted strings are equal, which is
done by the locally scoped `allEq`. Using `zipWith`, it compares each element
in the list with its successor, resulting in a list of `Bool`s. We use the
`and` function to fold the list into a single `Bool`: True iff all the elements
are True, and False otherwise.

## Commentary
This example showcases Haskell's laziness really nicely. Aside from the
obvious example of our use of infinite lists, there's also the fact that
`matching` will only check the multiples it needs to. That is, if 2x and 3x
aren't permutations, the `and` function will return False without forcing
evaluation of the remaining multiples, and we'll move on to the next integer.
This saves us from performing a bunch of needless sorts.

There's a couple weak points in the code, however. `firstOf` will crash if
called on an empty list, or a list with no elements that satisfy the input
predicate. Similarly, `allEq` will crash on empty lists. That is, both
`firstOf` and `allEq` are partial functions (defined on a subset of the
possible input).

I don't really see these as problems in such a simple program, but if these
functions were components in a larger software system, I would take the time to
make them into total functions (lest I run into the bug later on).
