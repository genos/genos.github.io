---
title: "CvRDT Exposition in Rust"
layout: post
---

# Programming with types for understanding

One of the best uses of the type system I've seen is the [Build systems a la carte](https://www.microsoft.com/en-us/research/publication/build-systems-la-carte/) paper.
In it, the authors use Haskell's types to outline and explore the problem domain in really novel ways.
I wanted to understand [CRDTs](https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type) more, so I made a thing.
It's nowhere near the level of _Build systems a la carte_, but I found it useful.

# Understanding CvRDTs

CRDTs are important to modern distributed programming; as the above Wikipedia article explains, they are

> data structure[s] which can be replicated across multiple computers in a network, where the replicas can be updated independently and concurrently without coordination between the replicas, and where it is always mathematically possible to resolve inconsistencies that might come up.

In reading about CRDTs, I came across the phenomenal paper [A comprehensive study of Convergent and Commutative Replicated Data Types](https://hal.inria.fr/inria-00555588/).
With this paper, Wikipedia, and some other references as my guide, I made a [Rust crate](https://docs.rs/cvrdt-exposition/) to understand state-based CRDTs (a.k.a. convergent replicated data types or CvRDTs).
I really like how this crate uses two traits with associated types to describe CvRDTs, fitting them into a common framework.

The first trait is for CvRDTs that can only grow, i.e. only add items.
Here's an slightly modified version of it; the code (on [GitHub](https://github.com/genos/cvrdt-exposition), with documentation on [docs.rs](https://docs.rs/cvrdt-exposition/)) has more:

```rust
pub trait Grow: Clone {
    type Payload: Eq;                                    // Internal state
    type Update;                                         // Message to update internal state
    type Query;                                          // Query message
    type Value;                                          // Query response

    fn new(payload: Self::Payload) -> Self;              // Create a new version of our CvRDT from Payload
    fn payload(&self) -> Self::Payload;                  // Retrieve Payload
    fn add(&mut self, update: Self::Update);             // Add item, mutating in-place
    fn le(&self, other: &Self) -> bool;                  // Is this ≤ other in semilattice's partial order?
    fn merge(&self, other: &Self) -> Self;               // Merge this and other into new CvRDT
    fn query(&self, query: &Self::Query) -> Self::Value; // Query to get some Value
}
```

Our rigorous typing does make for one difference from the aforementioned [paper](https://hal.inria.fr/inria-00555588/) and Wikipedia.
When implementing [`GCounter`s](https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type#G-Counter_(Grow-only_Counter)) and [`PNCounter`s](https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type#PN-Counter_(Positive-Negative_Counter)), we no longer have an unspecified `myId()` function giving the current node's index.
In order for the `Payload` type to fully specify their internal state, these classes require that index be part of the payload.

Other CvRDTs can also shrink, i.e. remove items.
Here's a slightly modified version of `cvrdt-exposition`'s trait:

```rust
pub trait Shrink: Grow {
    fn del(&mut self, update: Self::Update);             // Delete item, mutating in-place
}
```

# Testing and Verification

We have the `Eq` bound on the `Payload` type for verification and testing.
In order to verify that my CvRDT implementations behave as required, i.e. that their merge functions are commutative, associative, and idempotent, I turned to the [`proptest`](https://crates.io/crates/proptest) crate for property-based testing, via `assert_eq!` calls.
I turned to Rust's macro system to generate some of these tests for me; see [`properties.rs`](https://github.com/genos/cvrdt-exposition/blob/main/src/properties.rs) in the GitHub source and the `test` sub-modules of the [implementation modules](https://github.com/genos/cvrdt-exposition/tree/main/src) for more.
