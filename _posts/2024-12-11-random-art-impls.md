---
title: "Different Implementations of a Tiny Programming Language"
layout: post
---

# Random Art, Revisited

In [a previous article]({% post_url 2023-06-02-random-art %}), we played around
with implementing a tiny programming language for generating random art.
In that article and the [accompanying
code](https://github.com/genos/Workbench/tree/main/ra), we implemented a trick
I learned from [this
post](https://users.rust-lang.org/t/is-there-a-better-way-to-represent-an-abstract-syntax-tree/9549/4),
specifically [`matklad`'s](https://matklad.github.io/) response.
In learning more about programming language implementation, I came across [this
wonderful post](https://www.cs.cornell.edu/~asampson/blog/flattening.html) by
Professor Adrian Sampson, which examines the effects of flattening the abstract
syntax tree of a similar little language.
I thought it'd be interesting to perform a similar analysis with our random art
language implementation, so I put together [a Rust
project](https://github.com/genos/Workbench/tree/main/ra-impls) with four
different implementations.

# Implementations

The first implementation is the most basic, where we represent our AST directly:

```rust
pub enum Expr {
    X,
    Y,
    SinPi(Box<Expr>),
    CosPi(Box<Expr>),
    Mul(Box<Expr>, Box<Expr>),
    Avg(Box<Expr>, Box<Expr>),
    Thresh(Box<Expr>, Box<Expr>, Box<Expr>, Box<Expr>),
}
```

As mentioned [last time]({% post_url 2023-06-02-random-art %}), it can be more
efficient store a single pointer (i.e. `Box`), which my code confusingly calls
the "branch" version (apologies, but I wanted the names to be in alphabetical
order and it was the best I could come up with).
In this version, the main `Expr` looks like

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

where each of the intermediate structures looks like `struct Mul(Expr, Expr)`, etc.

After that, we move to a flat version similar to the one explored in Dr.
Sampson's [post](https://www.cs.cornell.edu/~asampson/blog/flattening.html).
In this, we go back to the basic structure, wherein we store (potentially)
multiple things in a single `Expr` variant:

```rust
enum Expr {
    X,
    Y,
    SinPi(ExprRef),
    CosPi(ExprRef),
    Mul(ExprRef, ExprRef),
    Avg(ExprRef, ExprRef),
    Thresh(ExprRef, ExprRef, ExprRef, ExprRef),
}
```

This time, though, the `ExprRef`s are instead indices, 32 bit unsigned integers, like `struct ExprRef(u32)`.
We then use a pool of expressions, `struct ExprPool(Vec<Expr>)`; during its
construction, every `ExprRef` index points to a valid index into this vector.
This controls memory layout even more tightly than our branch version, though
interpretation still
[uses](https://github.com/genos/Workbench/blob/main/ra-impls/src/flat.rs#L72)
an additional vector of the same length.

The final version follows [this
comment](https://old.reddit.com/r/ProgrammingLanguages/comments/mrifdr/treewalking_interpreters_and_cachelocality/gumsi2v/)
by [Bob Nystrom](https://craftinginterpreters.com/), modifying our flat version
to implement a stack-based bytecode interpreter.
Every subexpression is interpreted before larger expression that uses it, and
the ordering of items is all that matters.
We can eschew any kind of pointing or indexing, and move directly to a stack of individual instructions:

```rust
enum Expr {
    X,
    Y,
    SinPi,
    CosPi,
    Mul,
    Avg,
    Thresh,
}
```

These are then stored in order in a `struct ExprPool(Vec<Expr>)` during
construction, and
[interpretation](https://github.com/genos/Workbench/blob/main/ra-impls/src/stack.rs#L66)
uses a single stack.

# Do you even bench(mark)?

For benchmarking, for each version we build up a random expression of "depth"
26 in the same fashion as last post, then evaluate that expression with a
pseudorandomly chosen `x` and `y`.
Reaching for the amazing [`hyperfine`](https://github.com/sharkdp/hyperfine) to
time our various approaches:

<img src="{{ "/public/ra_shell.png" | absolute_url }}" alt="Hyperfine command for generating benchmarks">

We get the following times:

<img src="{{ "/public/ra_bench.png" | absolute_url }}" alt="Plot of benchmark times by command">

Interestingly, there's a large time improvement in moving from the basic to the branch version alone!
From there, we get further improvements moving to the flat and stack versions,
though the savings are less dramatic.

To keep myself honest, I also double-checked that versions yield equivalent
results (at least, up to a certain depth) via some [property-based
tests](https://github.com/genos/Workbench/blob/main/ra-impls/src/main.rs#L44-L70)
with the help of the exceptional [`proptest`
crate](https://github.com/proptest-rs/proptest).

# Coda & Code

As always, it was fun and interesting to take the lessons learned in reading and see their impacts in practice.
Please feel free to check out [the code](https://github.com/genos/Workbench/tree/main/ra-impls) for more details!
