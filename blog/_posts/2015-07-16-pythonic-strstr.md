---
layout: post
title: Pythonic strstr()
---

Wow, my strategy of writing a blog post every day failed *miserably*. I suppose
I didn't want to keep writing posts about PE problems, but also couldn't think
of anything more substantial to blog about, and with that indecision I simply
abandoned the task. But no more!

Over the past two months, I've been searching for a job here in SF. My parents'
house has afforded me a much longer period of time to search, and the
opportunity to be a bit picky. I've also gotten a lot of exposure to the
interview process, which is great, since within the ivory tower of academia I
learned absolutely nothing about how interviews actually work in this industry.

******

One of the questions I got on a phone screen was the standard `strstr()`
question:

> Write a function `strstr()` that accepts two strings: a "source" and a
> "pattern". Return the index of the first occurrence of "pattern" within
> "source", or -1 if it never occurs.

...and some examples of use:
> `strstr("banana", "na") => 2`

> `strstr("evening", "grr") => -1`

The name `strstr` refers to a standard C library function (from `string.h`).
It's also likely available in any other language's standard library (`str.find`
in Python, `String.indexOf` in Java, etc.). It's also a [well-researched
area](https://en.wikipedia.org/wiki/String_searching_algorithm), with plenty of
existing solutions that will outperform any solution I could come up with
while talking with a stranger over the phone.

So when I'm asked a question like this, its more than likely that the
interviewer just wants to see how I solve a problem. Sure, I could regurgitate
Knuth-Morris-Pratt or Boyer-Moore, but that would just show that I memorized an
algorithm. (Then again, that would probably be pretty impressive. Maybe I'll
try that on a phone screen.) I tend to reach for the naive solution.

I was coding in python for this interview, and wrote up something like the
following:

``` Python
def strstr(source, pattern):
    for i in range(len(source) - len(pattern) + 1)
        foundMatch = True
        for j in range(len(pattern)):
            if source[i+j-1] != pattern[j]:
                noMatch = False
                break
        if foundMatch:
            return i
    return -1
```

Hideous. The whole time I worked through that solution, I was well-aware of how
ugly any un-Pythonic it was. I blame interview nerves.

How can we make this better? One way is to stop comparing the substrings
character by character and instead compare the strings directly. We can extract
the substring from `source` by using Python's list slicing.

``` Python
def strstr(source, pattern):
    for i in range(len(source) - len(pattern) + 1)
        slice = source[i:i+len(pattern)]
        if slice == pattern:
            return i
    return -1
```

Much better already. However, I still have a problem with the way that the
slices are created. At each iteration of the loop, we create a new slice and
see if it matches -- that's fine, but its a little difficult to tell at first
glance. As [Rich Hickey would
say](http://www.infoq.com/presentations/Simple-Made-Easy), we're "complecting"
two things:

* how we get the stuff we're looping over, and
* what we do with the stuff we're looping over.

I prefer this:

``` Python
def strstr(source, pattern):

    starting_points = len(source) - len(pattern) + 1
    slices = [source[i:i+len(pattern)] for i in range(starting_points)]

    for i, slice in enumerate(slices):
        if slice == pattern:
            return i
    return -1
```

The slices are now in their own little list comprehension, and we worry about
checking them in the loop below. We've moved a little from an *imperative*
solution to a *declarative* one (though we still worry about "how" within the
list comprehension).

There's one subtle point that makes this new solution operate differently: the
list comprehension will compute all the slices and store them in memory before
we loop through them. If our `source` string is really long, this could be
really expensive in terms of memory usage.

But we can fix this! All we have to do is replace the parentheses with
brackets:

```Python
slices = (source[i:i+len(pattern)] for i in range(starting_points))
```

With this, we've ditched our list comprehension in favor of a *generator*
comprehension. Now, `slices` will be "lazy" and only compute a slice when we
ask for one inside the loop. If the slice doesn't match, it'll get garbage
collected once we move to the next iteration. This all makes our memory
footprint a lot lighter.

******

There. I've vindicated myself after writing such a horrible solution to the
problem during the phone screen. Not only that, I think this is a great example
of how to make our code "Pythonic" by focusing on declarative constructs. It's
still a naive, O(n^2) solution, but it's probably the best and cleanest one you
could give for this question within the span of a phone screen.
