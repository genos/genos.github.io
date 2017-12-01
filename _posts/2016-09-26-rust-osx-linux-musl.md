---
title: "Cross Compiling Static Rust Binaries for Linux on OS X"
layout: post
---

Recently I wrote a small [`Rust`](http://rust-lang.org) executable for work.
I wrote it on my Apple laptop, but wanted to run it on our Linux servers.
This note hopes to document how I did it so that I and others may repeat my
experience.

_Note: I only managed to pull this off because of the helpful folks on the
`#rust` and `#rust-internals` IRC channels. Thanks everyone!_

# Set up
There are plenty of tutorials out there for getting started with `Rust`, though
it's tough to beat [the official
book](https://doc.rust-lang.org/nightly/book/).
In order to get a working environment going, I recommend using
[rustup](https://www.rustup.rs/).

First, create a new binary project with `cargo`:

```bash
cargo init --bin my_proj
cd my_proj
```

In order to get stuff working later, use the `nightly` branch of `Rust`:

```bash
rustup override set nightly
```

Write your project as usual, editing `src/main.rs` to your heart's content.

# `musl`

In order to get your executable to run on the Linux server, you'll need to
compile & link it for that platform.
I've had luck creating an *entirely* static executable using `musl`, a rewrite
of the `C` standard library intended for smaller, embedded targets.
To get the necessary tools installed on OS X, use `homebrew` with [this helpful
tap](https://github.com/FiloSottile/homebrew-musl-cross).

Once you've got the linker & other compilation tools installed (as the above
link says, that'll take a while), you'll need to let `cargo` know to use them
when compiling for the Linux target.
Create a subdirectory to the `my_proj` directory called `.cargo`; then in
`.cargo/config`, write the following

```toml
[target.x86_64-unknown-linux-musl]
linker = "x86_64-linux-musl-gcc"
```

You'll also need to have `rustup` specify the target:

```bash
rustup target add x86_64-unknown-linux-musl
```


# Compile & Deploy
To compile your executable for the server, run

```bash
cargo build --release --target=x86_64-unknown-linux-musl
```

Copy it to the server and you're good to go:

```bash
scp target/x86_64-unknown-linux-musl/release/my_proj ⟨your_server_name⟩:
```

# UPX (added 2016-10-17)

If you're concerned about the [binary
size](https://www.rust-lang.org/en-US/faq.html#why-do-rust-programs-have-larger-binary-sizes-than-C-programs)
of your new executable, check out [`UPX`](https://upx.github.io).
After installing it on my laptop via `brew install upx`, I ran `upx -9` on an
executable created with the above instructions.
While the executable was an overly simplistic example, `upx` compressed it down
to _34%_ of the original size.
Even if you don't care about the size of the binary once it's on the server, it
at least made the `scp` go faster.
