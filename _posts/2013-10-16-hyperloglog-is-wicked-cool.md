---
title: "HyperLogLog is Wicked Cool"
layout: post
use_katex: true
---

# How to count without counting
I've been messing about with a few different things lately, one of which is the
[HyperLoglog
algorithm](http://algo.inria.fr/flajolet/Publications/FlFuGaMe07.pdf).
Others have
[already](http://blog.aggregateknowledge.com/2012/10/25/sketch-of-the-day-hyperloglog-cornerstone-of-a-big-data-infrastructure/)
[written](http://blog.aggregateknowledge.com/tag/hyperloglog/)
[about](http://metamarkets.com/2012/fast-cheap-and-98-right-cardinality-estimation-for-big-data/)
[this](http://static.googleusercontent.com/external_content/untrusted_dlcp/research.google.com/en/us/pubs/archive/40671.pdf)
extensively, so I have nothing terribly new to contribute; I just wanted
briefly discuss what it is and show some code.

# In Brief
While it's a little [zen](http://www.101zenstories.com/index.php?story=21)
sounding, HyperLogLog is a way of counting the number of unique items---say,
words in a file---without actually counting them.
That is, it gives a (rather good) estimate of the cardinality of the set of
elements in a given multiset while taking up much less memory than
instantiating the whole set (or a hashed version thereof), and it requires only
a single pass through the data with some smaller post-processing on the end for
a runtime of \\( O(N) \\).

# The larger picture
I think HyperLogLog is
[wicked](http://bangordailynews.com/2012/04/13/living/everybodys-heard-about-the-maine-words/)
cool.
More generally, though, I think it's a great example of a type of algorithm
that deserves more exploration, especially in this era of [big
data](http://memegenerator.net/instance/34328226): algorithms that trade
resource efficiency for approximate answers.
A big part of what's so cool about HyperLogLog is that the authors of
[this](http://algo.inria.fr/flajolet/Publications/FlFuGaMe07.pdf) paper give a
highly mathematical and probabilistic description of the possible error and
then mitigate that in the final version of the algorithm.
It's very impressive.

# Code
For a (slapdash?) C++11 version of the final algorithm, head
[here](https://github.com/genos/Programming/blob/main/workbench/hyperloglog.cc).
