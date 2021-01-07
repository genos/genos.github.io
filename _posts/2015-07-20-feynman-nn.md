---
title: "Learning By Building: A Simple Neural Network Implementation"
layout: post
---

# Small baby, no sleep make Graham a dull blogger
Apologies, readers, for my long absence.
My family grew this last year, so I've had less time for writing.

# Feynman's Blackboard
On [Richard Feynman's
blackboard](https://twitter.com/edwardtufte/status/566018453169913856) at the
time of his death was [the
quote](https://en.wikiquote.org/wiki/Richard_Feynman) *"What I cannot create, I
do not understand."*
Powerful words, and I definitely think there's some truth to them.

Recently I came across a popular, succinct
[post](http://iamtrask.github.io/2015/07/12/basic-python-network/) on
implementing a [neural
network](https://en.wikipedia.org/wiki/Artificial_neural_network) in Python.
I realized that although I've used them before and had a cursory understanding
of how they worked under the hood, I didn't really *grok* them; I had always
turned to a library.
Admittedly, turning to a well-developed library is exactly what one should do
in production, but for my own edification I decided to put together [a Python
implementation](https://github.com/genos/Programming/blob/main/workbench/nn.py)
of my own.
This implementation is rather simplistic, but it has three features I like:

1. It's simple enough that someone as sleep deprived as myself can
   understand it.
2. It feels Pythonic; it relies only upon `NumPy` and has an API similar to
   `sklearn`'s classifiers
3. It can handle an arbitrary number of hidden layers of arbitrary size.

# Lesson Learned
As awesome as neural nets are, as many varieties and variations there are, and
as amazing as some of the
[applications](http://googleresearch.blogspot.com/2015/06/inceptionism-going-deeper-into-neural.html)
may be, I discovered that at their base, they're rather simple.[^1]
Neural networks boil down to iterated linear algebra---which totally makes
sense, since they're being implemented on GPUs all over the place.
Though I wouldn't recommend going back to basics all the time (especially if
you're on a deadline), I'm definitely glad I took the time to work this
through.

[^1]: The vanilla version is, anyway; other variations are admittedly more complex.
