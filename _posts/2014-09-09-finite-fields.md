---
title: "Finite Fields: Surprisingly Tricky to Implement Well"
layout: post
use_katex: true
---

# Apologies
Sorry for taking so long to post again, fearless reader; I've recently moved to
a new house, so most of my outside-of-work-energy has been spent dealing with
all that entails.
This also means that this post is probably not as polished as I may have hoped.

# Intro
While attempting to port some academic code to a, shall we say, cleaner setup
for work, I had some trouble with implementing a piece of somewhat low-level
code that works with finite fields.
Finite fields are important, and the algorithms involved aren't too terribly
difficult to understand, as these things go.
That being said, I find that they're surprisingly tricky to implement well.

# Finite Fields are Important
Finite fields are interesting pieces of math in their own right, and involve
interesting topics and ideas like Euclid's Algorithm and Galois Theory.
They're quite important for other work, too, since they tend to form the
foundation upon which other ideas and structures build; akin to, say, how high
school geometry talks about points made of pairs of real numbers.

Finite fields have quite important real-world applications, too.
For one example, they form the underlying primitives upon which much of modern
cryptography is based; bitcoin wouldn't be possible without finite fields,
since it uses
[ECDSA](https://en.bitcoin.it/wiki/Elliptic_Curve_Digital_Signature_Algorithm).

# With that said, they're hard to implement
Finite fields are slightly tricky to implement, though, because they are
basically parameterized by some information (the prime characteristic in the
prime field case, and the polynomial modulus in the general case).
Prior to doing any computation within a given field, you have to know exactly
which field you're dealing with.

Suppose you wanted to write some software to compute in a prime field; what
approach would you take to storing/initializing the characteristic?
Here are some existing options:

1. [NTL's](http://www.shoup.net/ntl/) approach: global initialization of
   finite field element type, `ZZ_p::init(p)`; this is what I used for [the
   code](https://github.com/genos/e2c2) portion of [my
   dissertation](https://github.com/genos/dissert_and_defense).
   [FLINT's](http://www.flintlib.org/) `f_q` stuff looks similar, though I
   haven't used it.
2. The [Haskell For
   Maths](https://hackage.haskell.org/package/HaskellForMaths-0.4.5/docs/Math-Core-Field.html)
   approach: use `newtype`s for some small examples (`F2, F3, F25`, etc.),
   including extension fields.
3. The [Finite](https://github.com/msakai/finite-field)
   [Fields](http://hackage.haskell.org/package/finite-field) Haskell
   package: type level numbers to hold the characteristic
4. The [Sage](http://www.sagemath.org/) approach: on the surface, there is a
   class within which you do the work, like `GF(17)(3 * 5)`.
   Behind the scenes, however, this is implemented by (depending on the version
   of Sage and the finite field in question) a package like NTL or FLINT (both
   mentioned above)
5. The elegant `with-modulus` Scheme macro from [this Programming
   Praxis](http://programmingpraxis.com/2009/07/07/modular-arithmetic/)
   exercise.

All of these approaches have their drawbacks, however:

1. NTL requires initializing the field globally *before* any usage.
   I had to chase down crashes early on in writing code for my thesis; it
   wasn't fun
2. This package only defines a few fields, not a general parameterized way of
   easily working over *any* prime field
3. This package is quite nice, but only works for prime fields; I have no idea
   how to work with, e.g., \\(\mathbb{F}_{2^8}\\) for
   [AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard).
4. This is nice on the surface, but that nice interface hides a *lot* of
   different ways of doing things.
   It also requires the field object to know about the element object and the
   element object to simultaneously know about its parent field object...which
  can lead to development headaches.
5. This is a great way to go, since it clearly marks what field you're working
   with and delimits the scope of those computations nicely---but how many
   people know how to effectively craft macros?

And all of these problems are only regarding working in a single field!
What about a tower of extensions, or working with other isomorphisms between
fields?

# Moral
Math is full of useful stuff, but it may not be readily apparent how best to
implement those things in a programming language.
In the finite field case, the issue (for me) boils down to effectively dealing
with the extra information that parameterizes our field.
And a lot of potentially useful mathematical objects similarly require knowing
some parameterizing information.
For instance, to work with, say, [points on binary Edwards
curves](https://github.com/genos/e2c2), there are several layers of this going
on: points are parameterized by their coordinates and the curve on which they
lie.
Curves and coordinates, in turn, are embedded in some field; in this case, an
extension field of \\(\mathbb{F}_2\\).
That extension field, as mentioned above, is parameterized by its
characteristic (2) and its modulus (some polynomial in \\(\mathbb{F}_2[x]\\)).
My dissertation code is one way of dealing with all of that detail, but I can't
help but wonder if there is a better way of going about it.
