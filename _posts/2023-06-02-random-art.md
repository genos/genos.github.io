---
title: "Random Art with a Tiny Language"
layout: post
use_katex: true
---

# PL Resources Everywhere

In [playing around with OCaml]({% post_url 2023-05-11-kompreni %}), I've
spent some spare time perusing more programming language resources.
There are a bunch out there, especially in this era of online learning; for
instance, Cornell has made some
[great](https://www.cs.cornell.edu/courses/cs6120/2020fa/lesson/)
[resources](https://www.cs.cornell.edu/courses/cs6110/2019sp/schedule.html)
available.
In looking specifically for more ML-ish[^1] flavored ones, I came across
[this](https://ucsd-progsys.github.io/cse130/homeworks/hw2.html) fun undergrad
homework set.
In the second problem, it asks you to implement a tiny (recursively
defined) language of expressions encoding functions from the unit square to the
unit interval, \\( [0, 1]^2 \mapsto [0, 1] \\).

```ocaml
type expr = VarX 
          | VarY 
          | Sine of expr 
          | Cosine of expr 
          | Average of expr * expr 
          | Times of expr * expr 
          | Thresh of expr * expr * expr * expr 
```

After you learn to build them up randomly, you can evaluate them across the
unit square to generate pictures—have each output value encode either a
greyscale value, or one of an RGB triple.
Note to undergrads: _please_ don't cheat off me; you'll only rob yourself of a
fun learning experience.

# No Peeking!

This was a fun exercise in OCaml, but I also found it instructive to tackle it
[in Rust](https://github.com/genos/Workbench/tree/main/ra) and compare.
Besides the superficial differences, Rust's concern with and ability to control
memory usage is apparent.
I defined the core enumeration thusly:

```rust
pub enum Expr {
    X,
    Y,
    SinPi(Box<Expr>),
    CosPi(Box<Expr>),
    Mul(Box<Mul>),
    Avg(Box<Avg>),
    Thresh(Box<Thresh>),
}
```

I renamed some variants to more accurately express their intent; e.g.
`SinPi(expr)` evaluates to \\( sin(\pi \cdot \\) `expr` \\(\\!)\\).
More importantly, I reworked the recursive variants to each contain (at most) a
single `Box` (i.e. unique pointer to a heap allocated value); for instance,
`Expr::Mul` contains a `Box<Mul>`, where `Mul` is defined as

```rust
pub struct Mul {
    e1: Expr,
    e2: Expr,
}
```

I took up this trick after
[learning](https://users.rust-lang.org/t/is-there-a-better-way-to-represent-an-abstract-syntax-tree/9549/4)
more about it from the inimitable [`matklad`](https://matklad.github.io/).
As the linked response says, this keeps the size of the base enum smaller.
In the source code, I've got a [property
test](https://github.com/genos/Workbench/blob/main/ra/src/lib.rs#L242-L245)
that asserts the size of an arbitrary[^2] `Expr` is exactly 16 bytes[^3]:

```rust
#[test]
fn size_of(e in arb_expr()) {
    assert_eq!(size_of_val(&e), 16)
}
```

If we instead went with a more direct translation of the OCaml code, like

```diff
diff --git a/src/lib.rs b/src/lib.rs
index 3b04a74..68c600c 100644
--- a/src/lib.rs
+++ b/src/lib.rs
@@ -7,9 +7,9 @@ pub enum Expr {
     Y,
     SinPi(Box<Expr>),
     CosPi(Box<Expr>),
-    Mul(Box<Mul>),
-    Avg(Box<Avg>),
-    Thresh(Box<Thresh>),
+    Mul { e1: Box<Expr>, e2: Box<Expr> },
+    Avg { e1: Box<Expr>, e2: Box<Expr> },
+    Thresh { e1: Box<Expr, e2: Box<Expr>, e3: Box<Expr>, e4: Box<Expr> },
 }
```

then this property test would no longer pass, as the enum needs to be large
enough to contain its largest variant.


# Pretty Pics Plz

Once I coded up the library, I also put together a small
[executable](https://github.com/genos/Workbench/blob/main/ra/src/main.rs) to
take in some parameters and generate greyscale or RGB pictures, similar to the
original exercise.
With that defined, I could run it a bunch of times using
[`jot`](https://www.oreilly.com/library/view/mac-os-x/0596003706/re254.html) to
generate random seeds and [`parallel`](https://www.gnu.org/software/parallel/)
to use multiple cores:

```rust
jot -w %i -r 20 0 1000000000 | parallel --bar ./target/release/ra -s {} -d 7 -r
```

Here are some gems I found combing through the PNGs:

<table>
    <tr>
        <td><img src="{{ "/public/126089461_7_512_512_rgb.png" | absolute_url }} " width="66" alt="RGB Image for seed=126089461, depth=7 width & height=512"></td>
        <td><img src="{{ "/public/208259458_7_512_512_rgb.png" | absolute_url }} " width="66" alt="RGB Image for seed=208259458, depth=7 width & height=512"></td>
        <td><img src="{{ "/public/347723950_7_512_512_rgb.png" | absolute_url }} " width="66" alt="RGB Image for seed=347723950, depth=7 width & height=512"></td>
    </tr>
    <tr>
        <td><img src="{{ "/public/765744456_7_512_512_rgb.png" | absolute_url }} " width="66" alt="RGB Image for seed=765744456, depth=7 width & height=512"></td>
        <td><img src="{{ "/public/983185677_6_512_512_rgb.png" | absolute_url }} " width="66" alt="RGB Image for seed=983185677, depth=6 width & height=512"></td>
        <td><img src="{{ "/public/161582962_7_512_512_rgb.png" | absolute_url }} " width="66" alt="RGB Image for seed=161582962, depth=7 width & height=512"></td>
    </tr>
    <tr>
        <td><img src="{{ "/public/176235008_7_512_512_rgb.png" | absolute_url }} " width="66" alt="RGB Image for seed=176235008, depth=7 width & height=512"></td>
        <td><img src="{{ "/public/61786666_7_512_512_rgb.png" | absolute_url }} " width="66" alt="RGB Image for seed=61786666, depth=6 width & height=512"></td>
        <td><img src="{{ "/public/627656271_7_512_512_rgb.png" | absolute_url }} " width="66" alt="RGB Image for seed=627656271, depth=7 width & height=512"></td>
    </tr>
</table>

[^1]: [ML](https://en.wikipedia.org/wiki/ML_(programming_language)) as in OCaml
    and SML, not machine learning.

[^2]: I _really_ like property-based testing, in this case with
    [`proptest`](https://docs.rs/proptest/latest/proptest/).

[^3]: Since
    [`size_of_val`](https://doc.rust-lang.org/std/mem/fn.size_of_val.html)
    returns the size of the pointed-to value in bytes, this test returns 16 on
    my 64-bit laptop; it will probably fail on a 32-bit architecture. Be sure
    to check out
    [this](https://fasterthanli.me/articles/peeking-inside-a-rust-enum) awesome
    article for more on Rust enums.
