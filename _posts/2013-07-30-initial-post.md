---
title: "Initial Post"
layout: post
use_katex: true
---
So... here's my first post.
I've been inspired to keep a digital notebook/record of sorts, mostly to keep
me honest about continuing my education; the theory is that if I have something
to keep me accountable, some place to post my work, then I'll keep working.

Since this is my first post, I'm going to use this space for two things:

1. as a play sandbox to see how things render
2. to dicuss some of the things I hope to write about

# Sandbox
My father the math-teacher-extraordinaire says that Euler's
identity \\( e^{i\pi} + 1 = 0 \\) is proof of the existence of God.
Here's some `Python` code that tries to verify this fact:
{% highlight python %}
from cmath import exp, pi
print(exp((0 + 1j) * pi) + 1)
{% endhighlight %}
Due precision issues, this probably won't come out to zero.
However, it sure looks darn close.

# Serious Note
My dissertation focused on applying a specific type of abstract algebra to
cryptography (for the technically inclined: the use of Edwards curves,
specifically binary Edwards curves, in elliptic curve cryptography; if you have
trouble sleeping, I can provide a copy of my dissertation).
As such, I have a soft spot for computational group theory, abstract algebra in
general, and cryptography.
Though I have no problem geeking out on these topics at the drop of a hat, I'm
hoping to broaden my horizons a bit---writing a dissertation is a bit draining.
I'm particularly interested in learning more probability, statistics, and
machine learning; though as of this writing I have a passing understanding of
these topics, I hope to dive into them much more deeply.
