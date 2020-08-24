---
title: "Find X[i] = i in an array (Programming Praxis)"
layout: post
use_katex: true
---

# Introduction
The inimitable [Programming Praxis](http://programmingpraxis.com), to which I
owe much of my programming abilities[^1],
[posted](http://programmingpraxis.com/2013/07/26/find-xi-i-in-an-array/) an
exercise that was seemingly simple but has some surprising depth to it.
Since it's been a little while since the problem was posted, I have few qualms
about discussing it here.
However, don't read on if you prefer to solve the problem yourself first.

# The question
Programming Praxis asks,

<blockquote>
<p>
Given an array <code>X[0 .. n - 1]</code> of integers sorted into ascending
order with no duplicates, find an array item that is also its index, so that
<code>X[i] = i</code>.
</p>
<p>
For example, <code>X[3] = 3</code> in the array shown below:
</p>
<p>
{% highlight text %}
i     0 1 2 3 4 5
X[i] -3 0 1 3 5 7
{% endhighlight %}
</p>
</blockquote>

# Official solutions
The first official solution is a simple linear search through the array; start
at index `0` and walk up until you either find an `i` such that `X[i] = i` or
hit the end of the array.
This solution, however, is dismissed in favor of a better one that uses a
binary search.
Since the array was presumed to be sorted, a binary search finds the answer in
logarithmic time instead of linear time.
The author points out that if you only stick with the linear answer in an
interview, chances are you won't make the cut.

# A different approach
I've been thinking about a different interview scenario in which you probably
still won't make the cut if you stick with the linear solution.
Suppose you're trying to claim competency in the [R statistical programming
language](http://www.r-project.org/).
If you submit the linear solution in R, you'd still fail (or at least, not
succeed with flying colors).
However, I think that the binary search approach is wrong here as well; much of
R depends on thinking over a whole collection at a time, through
[vectorization](http://en.wikibooks.org/wiki/R_Programming/Control_Structures).
For example, one can quickly ask which elements of the vector `V` are less than
ten by typing `V < 10`.

Here's my R solution in full:[^2]
{% highlight r %}
X <- c(-3, 0, 1, 3, 5, 7)
X[X == seq(length(X)) - 1]
{% endhighlight %}

The first line simply assigns the example vector to the variable `X`, as we
might expect; the `c()` function combines or concatenates its arguments.
Next, let's talk about the sequence function; `seq(n)` returns the vector `1 2
3 ... n`; this means that `seq(length(X))` gives the sequence of integers from
`1` to the length of our vector `x`, and subtracting one from it yields `0 1 2
...  length(X) - 1`.
That subtraction was the first time we took advantage of R's implicit loops;
the second time comes when we compare `X` to this new vector we've created.
The expression `X == seq(length(X)) - 1` yields a boolean vector which is
`True` wherever `X[i] == i` and false wherever they differ, just as we wish.
Finally, the outer `X[]` returns those entries in `X` indexed by the inner
expression, i.e. only those `X` values corresponding to a `True` value.

Though I'm not an R expert, I think this is pretty idiomatic R programming and
would be a more acceptable answer in an interview geared toward demonstrating
knowledge of the statistical programming language---I'm trying to learn,
though, so if anyone has a better solution, please let me know!
What's more, it will probably end up being faster than the logarithmic binary
search just due to the way R is designed; it's optimized for these vectorized
expressions.
While explicit looping and conditionals can be done in R, they tend to be much
slower.
If you reach the point where the logarithmic solution in slower R code catches
up with the linear implicit looping one, chances are your problem is big enough
that R isn't the language to use anyway.

# Brevity and elegance

I also offered a solution in the [J programming
language](http://en.wikipedia.org/wiki/J_programming_language) which operates
in the same way but is probably harder to read---harder for the uninitiated, at
least.
Per the Wikipedia page, J is an array-oriented functional programming language
that grew out of Kenneth Iverson's
[APL](http://en.wikipedia.org/wiki/APL_(programming_language)).
As a mathematician, I find these languages fascinating; they endeavor to be
both "executable math" and _better_ math.
I'll try to explain both of these points a bit, but I'm sure I won't do them
justice.

As far as "executable math" is concerned, suppose we wanted, say, the sum of
the first 17 natural numbers.
In plain ol' C, we'd go with something like this:

{% highlight c %}
int sum = 0;
for (int i = 0; i < 17; ++i) {
    sum += i;
}
printf("%d\n", sum);
{% endhighlight %}

Expressing the same thing in J is much more declarative.
In J, `i.17` is the first `17` natural numbers.
Next, `+/` is sum, since it's plus
[folded](http://en.wikipedia.org/wiki/Fold_(higher-order_function)) over its
argument.
Hence, `+/i.17` is the sum we want; compare that to the brevity of
\\[ \sum_{i = 0}^{16} i \\]
The J code says what the C loop tries to say, but much more succinctly; there's
very little in the way of taking the mathematical expression---which in
un-typeset LaTeX reads something like "the sum from `i = 0` to `16` of
`i`"---and having the computer execute it.

As for the "better math" claim, APL originally grew out of a mathematical
notation that Kenneth Iverson created.
I won't go into much detail here, but will give one example stolen from [this
presentation](http://www.vaxman.de/publications/apl_slides.pdf).
In APL, all evaluation proceeds from right to left; there is no order of
operations, so no need for parentheses to insure that, e.g., an addition
happens before a multiplication.
As such, Horner's rule for polynomial evaluation of
\\( a + bx + cx^2 + dx^3 + ex^4 \\)
goes from this:
\\[ a + x \cdot(b + x \cdot (c + x \cdot (d + x \cdot e))) \\]
to this:
\\[ a + x \cdot b + x \cdot c + x \cdot d + x \cdot e \\]
In his 1979 Turing Award Lecture titled [Notation as a Tool of
Thought](http://www.jsoftware.com/papers/tot.htm), Iverson makes a strong
argument for improving mathematical and programming notation by combining them
to better serve our needs.
Though I may not be entirely sold on this just yet, it certainly is a very
interesting idea and I will definitely explore it more.

# A J solution
Now for my J solution to the original question:
{% highlight text %}
(I.@:=i.&#) _3 0 1 3 5 7
{% endhighlight %}
or, if we assign the array the name `X`,
{% highlight text %}
X =. _3 0 1 3 5 7
I.X=i.#X
{% endhighlight %}
I won't discuss the first one here (to be honest, I'm very new to J and don't
understand it well enough).
However, the second one should look an awful lot like the R solution we saw
earlier.
Taking it in pieces, `#X` returns the length of the vector `X`.
As we saw above, `i.` counts from `0` up to one less than its argument, so
`i.#X` yields the vector \\(\langle 0, 1, 2, \cdots \\# X - 1\rangle\\)
Next, `X=i.#X` returns a vector of ones and zeros indicating where `X` is equal
to this vector (i.e. where `X[i]` equals `i`).
Finally, the operator `I.X` uses that boolean vector to select values from `X`,
much as `X[]` did in the R solution.

# Summing up
I think Programming Praxis is right in saying that in a typical programming
interview, the linear approach to this problem just won't cut it.
However in a stats-geared interview, I think the logarithmic answer might be
less than stellar, too.
Though my R solution doesn't make use of the fact that `X` was supposed to be
sorted (nor does my J solution), it is much more idiomatic and demonstrates a
better understanding of what R is supposed to do and what it does well.
Both of my solutions also have the added advantage of finding all possible
answers in an array, not just one.

# Postscript
In the discussion section at the end of [Formalism in
Programming Languages](http://www.jsoftware.com/papers/FPL.htm), Iverson
answers a question from Dijkstra that sounds very familiar:

> _Dijkstra_: How would you represent a more complex operation, for example,
> the sum of all elements of a matrix __M__ which are equal to the sum of the
> corresponding row and column indices?
>
> _Iverson_:[^3] \\( ++/(\mathbf{M} = \iota^1 {\Box \atop +} \iota^1) //
> \mathbf{M} \\)

[^1]: Such as they are.
[^2]: I've changed all `xs` in my submitted solution to `X` to be more in line with the question.
[^3]: The "box atop plus" symbol above isn't quite right, but regretfully I don't know how to make the appropriate APL symbol in the limited LaTeX afforded to me by MathJax & \\( \KaTeX \\).
