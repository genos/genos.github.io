---
title: "Group Theory AND Statistics"
layout: post
---

# Diaconis does something cool, video at 11
[Persi](http://statweb.stanford.edu/~cgates/PERSI/)
[Diaconis](http://en.wikipedia.org/wiki/Persi_Diaconis) is an interesting
character. He dropped out of high school at 14 to become a professional
magician, got a Ph.D. from Harvard, won a MacArthur "genius grant," and is now
a mathematician and statistician at Stanford University.

He also does interesting math; today I'll talk about an example from
a paper[^1] of his that I found particularly clever.

# Counting voter rankings
Suppose we are holding an election between five candidates, and we ask voters
to rank all five in order of preference. We'd like to take those rankings of
the five candidates and turn it into a table where the *(i, j)* entry is the
number of voters who ranked candidate *i* in place *j*.

Note that a ranking of five candidates can be thought of as a permutation of
length five, an element of the group \\( S_5 \\). That is, the permutation
\\((5 4 3 2 1)\\) corresponds to ranking candidate 5 first, candidate 4 second,
etc. Next, we use the usual linear representation of \\( S_5 \\); for any given
permutation \\( \pi \\), let \\( \rho(\pi) \\) be the 5 x 5 matrix with *(i,
j)* entry equal to one if \\( \pi(i) = j \\), zero otherwise.

Define a function \\( f: S_5 \rightarrow \mathbb{N} \\) such that
\\( f(\pi) \\) is the number times the vote \\( \pi \\) was cast. Hence, if 10
people all voted \\((5 4 3 2 1)\\), then \\( f( (5 4 3 2 1) ) = 10 \\). Now,
take the Fourier Transform
\\[
    \hat{f} = \sum_{\pi} f(\pi) \rho(\pi)
\\]
Note that \\( \hat{f} \\) adds up the matrix for each vote times the number
of times that vote appeared; this means that \\( \hat{f} \\) gives us
a "table" (really a matrix) where the *(i, j)* entry is the number of times
voters ranked candidate *i* in place *j*. This is exactly what we want!

# Pithy Python program performs promptly
Here's some `Python` code that computes this.

First off, some useful imports.

{% highlight python %}
from collections import Counter
from itertools import permutations
import numpy as np
{% endhighlight %}

Next, we define our representation function \\( \rho \\).

{% highlight python %}
def rho(p):
    return np.matrix([np.eye(len(p), dtype=int)[:, i] for i in p])
{% endhighlight %}

After that, some (wicked short!) code to compute the transform
\\( \hat{f} \\).

{% highlight python %}
def fft(ps):
    c = Counter(tuple(p) for p in ps)
    return sum(rho(p) * c[p] for p in c)
{% endhighlight %}

The following code also works, but it doesn't technically follow the definition
above since it hides \\( f \\).

{% highlight python %}
def fft2(ps):
    return sum(map(rho, ps))
{% endhighlight %}

A quick function to normalize a table of counts to percentages instead:

{% highlight python %}
def table(xss):
    return 100 * xss / np.sum(xss, axis=1)
{% endhighlight %}

And a quick check to see it in action:

{% highlight python %}
if __name__ == '__main__':
    N = 5
    S_N = np.array(list(permutations(range(N))))
    ps = S_N[np.random.randint(np.factorial(N), size=500)]
    fhat = fft(ps)
    print("Permutations:\n{0}\n\nFFT:\n{1}\n\nPercentages:\n{2}".format(
            ps, fhat, table(fhat)))
{% endhighlight %}

# Fin.
I'm very excited about this; using group theory to better understand statistics
and machine learning is exactly what I've been hoping to do since I finished
grad school.

[^1]: "Applications of Group Representations to Statistical Problems," 1990; available [here.](http://www.mathunion.org/ICM/ICM1990.2/Main/icm1990.2.1037.1048.ocr.pdf)
