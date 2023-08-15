---
title: "A Rusty Quantum Interpreter"
layout: post
use_katex: true
---

# The Original

Robert Smith, a.k.a. `stylewarning`, has a [lovely blog
post](https://www.stylewarning.com/posts/quantum-interpreter/) that walks
through implementing an interpreter for a "general-purpose quantum programming
language called \\( \mathscr{L} \\)."
In only 150 lines of Common Lisp, [the
implementation](https://github.com/stylewarning/quantum-interpreter/) is
featureful, self-contained, and a delight to read.

# Imitation is the highest form of flattery

At first I was content with only reading the post and code, but then [I
saw](https://twitter.com/Sheganinans/status/1683875551587741703) someone else
had put together an OCaml implementation as well.
The game was afoot!

# Carcinization

Eventually I put together a [Rust
implementation](https://github.com/genos/Workbench/tree/main/quantum-interpreter).
This version weights in at more than twice the line count as the
original---even though it relies on
[`ndarray`](https://docs.rs/ndarray/latest/ndarray/index.html) for linear
algebra instead of implementing it by hand!
However, it does have a couple of features not present in the original (or
OCaml) implementation(s):

- constructing a `Machine` takes an unsigned 64-bit `seed` for repeatable PRNG
  behavior; and
- with the help of the [`peg`
    crate](https://docs.rs/peg/latest/peg/), I added parsing, so one can pass
    `"H 0\nMEASURE"` as a string and get an interpretable quantum program.

# Code or it didn't happen

The implementation is
[here](https://github.com/genos/Workbench/tree/main/quantum-interpreter) in my
catch-all "workbench" repository.
It's definitely _not_ production-worthy code; it's got an `assert!` that will
panic if not met, and it uses `expect` to pave over some errors.
Please feel free to take a look and let me know what you think!
