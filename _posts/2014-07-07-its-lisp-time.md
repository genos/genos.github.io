---
title: "It's Lisp Time"
layout: post
---

# Porting academic code is fun, I swear
I've been chewing on a rather tough programming problem at work recently
involving porting code from a collection of different languages to a C++11
production-ish-looking thing.
Really, the project is just to get a cohesive version of the interesting (but
entirely academic, not remotely production-ready) code from a published paper.
Even with the [previously mentioned niceties]({% post_url 2013-08-12-python-to-cxx %})
that C++11 has to offer, it's still rough going.
And I've realized that it's not just because the code is, *ahem*, interesting.
It's that my tools are a bit limiting.

## Everything else is too limit{ed,ing}

As it turns out, I'm finding more and more that the things people say singing
the praises of Scheme and Common Lisp might be on to something.
Take, for instance, Peter Seibel's excellent book [Practical Common
Lisp](http://www.gigamonkeys.com/book/).
In Chapter 9, he puts together a very nice unit testing framework; it's fully
commented and documented, has great reporting that tells you exactly what went
wrong, and offers a nice little domain specific language for writing tests.
The really awesome part: with comments, indentation, and line breaks, all this
happens in **just 26 lines of code**.

Now, I know there are other [small unit testing
frameworks](http://www.jera.com/techinfo/jtns/jtn002.html) out there,[^1] but
this is just amazing.
And it's just the tip of the iceberg.

# All hail the mighty parentheses
My motivation for wanting more Lisp & Scheme in my coding is twofold, I guess;
first, my work programming, while fun and useful, is constraining in some
senses.
Pretty much any project I write will be either in C, C++, or Python.
If it's a very data-science-y project, I might get to[^2] use R.
Those languages are great, and they all have their uses, but there's other
stuff out there, some of it [wildly more
powerful](http://www.paulgraham.com/avg.html).
So the first reason is because I want to use more interesting and powerful
tools.[^3]

The second reason is my own edification.
Scheme and Lisp have a decently long history (as far as programming languages
are concerned), and definitely have a library of great work for me to ingest.
There's the [previously mentioned]({% post_url 2013-10-04-sicp-is-so-good %})
[SICP](http://mitpress.mit.edu/sicp/full-text/book/book.html) for Scheme.
I also hope to spend more time on [Programming
Praxis](http://programmingpraxis.com/), though my attendance has been rather
poor lately.
For Lisp, I hope to read Paul Graham's [On
Lisp](http://www.paulgraham.com/onlisp.html) and Peter Norvig's
[PAIP](https://en.wikipedia.org/wiki/Paradigms_of_AI_Programming%3A_Case_Studies_in_Common_Lisp).
If I'm feeling particularly adventurous, I hope to read Doug Hoyte's [Let over
Lambda](http://www.letoverlambda.com/).
I've also been intrigued by Clojure recently,[^4] and have been paging through some
of Fogus and Houser's [Joy of Clojure](http://joyofclojure.com/).

# Comments/Questions/etc.?
Does it look like I've missed anything, dear reader(s)?
Please let me know if you have other recommendations, etc. for the road to Lisp
enlightenment.

[^1]: I've actually used `minunit` before, and it's great for C code. The point is, however, the framework from Seibel's book is simultaneously more powerful, more expressive, and easier to write tests in (to strand a preposition).

[^2]: Be forced to?

[^3]: And on one work project I was forced (for reasons I'll not go into, but not of my choosing) to use *Perl*. Kidding aside, it can be a useful tool for some things, but I assure you that what I was asked to build with it is **not** one of those things.

[^4]: I'll admit I'm a bit reticent about using the JVM, but Clojure has some nice features, so I'm trying to keep an open mind. I'm particularly interested in [Incanter](http://incanter.org/) as a possible replacement for R and Python for data work.
