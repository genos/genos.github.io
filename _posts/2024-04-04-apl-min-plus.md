---
title: "Tropical Semiring in Dyalog APL"
layout: post
use_katex: true
---

# A few of my favorite things

The [Tropical (min-plus)
semiring](https://en.wikipedia.org/wiki/Tropical_semiring) is one of my
favorite examples of how changing one's perspective can make difficult problems
much simpler.[^1]
In this semiring, instead of using the usual addition and multiplication,
we replace them with minimum and addition, so, for instance, \\( 1 \oplus 2 = 1
\\) and \\( 3 \otimes 2 = 5 \\).
I've [written]({% post_url 2023-05-11-kompreni %}) and [presented]({% post_url
2018-02-20-algebraic-structure %}) about [this]({% post_url
2022-09-13-ascb-in-rust %}) before.

# Links from the farm

Recently, someone on the [APL Farm](https://aplwiki.com/wiki/APL_Farm) posted
[an excellent
article](http://blog.sigfpe.com/2009/05/three-projections-of-doctor-futamura.html)
by `sigfpe` (AKA [Dan](https://mathstodon.xyz/@dpiponi)
[Piponi](https://twitter.com/sigfpe)), which prompted me to post a
[different](http://blog.sigfpe.com/2007/06/how-to-write-tolerably-efficient.html)
article where they work with the min-+ semiring.
I also posted [this article](https://r6.ca/blog/20110808T035622Z.html) by
Russell O'Connor, which is one of my all-time favorites.
In it, there's an example where the distances between nodes in a graph gets
turned into a Tropical matrix, and algorithms for computing shortest distances
become iterated linear algebra over this different semiring.
And then I wanted to work in APL instead of Haskell...

# An open dialogue

After a bit of head scratching, I realized that [Dyalog
APL](https://www.dyalog.com/) is really good for manipulating matrices over the
min-+ semiring.
Taking `exampleGraph2` from O'Connor's blog post, and letting \\( 10^{20} \\)
stand in for \\( \infty \\)[^2]:

```apl
inf←1e20
dist←6 6⍴0 7 9 inf inf 14 7 0 10 15 inf inf 9 10 0 11 inf 2 inf 15 11 0 6 inf inf inf inf 6 0 9 14 inf 2 inf 9 0
```

we have the following distance matrix

```apl
0.0E0  7.0E0  9.0E0  1.0E20 1E20 1.4E1
7.0E0  0.0E0  1.0E1  1.5E1  1E20 1.0E20
9.0E0  1.0E1  0.0E0  1.1E1  1E20 2.0E0
1.0E20 1.5E1  1.1E1  0.0E0  6E0  1.0E20
1.0E20 1.0E20 1.0E20 6.0E0  0E0  9.0E0
1.4E1  1.0E20 2.0E0  1.0E20 9E0  0.0E0
```

With this matrix, representing distances between vertices in our graph, we can
use the APL matrix product operator `.` _with different operations_ to perform
our calculations; instead of `+` and `×` for addition and multiplication, we
turn to `⌊` (min) and `+`; then `dist ⌊.+ dist` gives us the two-hop distances:

```apl
      dist ⌊.+ dist
 0  7  9 20 23 11
 7  0 10 15 21 12
 9 10  0 11 11  2
20 15 11  0  6 13
23 21 11  6  0  9
11 12  2 13  9  0
```

We can make this made more succinct with `⍨`, telling the interpreter to apply our
function with `dist` as both arguments:

```apl
      ⌊.+⍨ dist
 0  7  9 20 23 11
 7  0 10 15 21 12
 9 10  0 11 11  2
20 15 11  0  6 13
23 21 11  6  0  9
11 12  2 13  9  0
```

I was very excited that I could write down the steps of the
Gauss-Jordan-Floyd-Warshall-Kleene algorithm in so few characters!
Moreover, iterating that step until convergence is similarly succinct using the
power operator `⍣`; we can keep running `⌊.+` until the output matches (`≡`) the input:

```apl
      ⌊.+⍨⍣≡dist
 0  7  9 20 20 11
 7  0 10 15 21 12
 9 10  0 11 11  2
20 15 11  0  6 13
20 21 11  6  0  9
11 12  2 13  9  0
```

I can't help but feel that this was the type of thing Iverson meant by
[notation as a tool of thought](https://www.jsoftware.com/papers/tot.htm).


[^1]: As Alan Kay [says](https://quoteinvestigator.com/2018/05/29/pov/), point
    of view is worth 80 IQ points.

[^2]: I tried this in [BQN](https://mlochbaum.github.io/BQN/) as well, since it
    has \\( \infty \\), but the linear algebra and fixed point iteration are
    nicer in APL.
