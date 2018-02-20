---
title: "Slides: Algebraic Structure, Computational Benefits"
layout: post
---

# Algebraic Structure, Computational Benefits

I recently gave a talk at work titled "Algebraic Structure, Computational
Benefits."
In it, I defined a few basic (as in fundamental, not as in easy) structures
like semigroups, monoids, and semirings, and I discussed the computational
benefits one can derive by making these structures explicit in code, like:

- the most generic of generic algorithmic code
- parallelism for free, a.k.a. associativity implies `Map-Reduce`
- associativity + commutativity ⇒ `Map-Shuffle-Reduce`

I also talked a little bit about one of my recent favorites, the
[_Tropical Semiring_](https://en.wikipedia.org/wiki/Tropical_geometry), and how
it turns shortest-distance problems in graph theory into iterated linear
algebra.

Should any of this spark your interest, slides are available [here.](/public/ascb.pdf)
