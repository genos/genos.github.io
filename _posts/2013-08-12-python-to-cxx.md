---
title: "Python to C++; a silly idea?"
layout: post
---

# Python type annotations
In the 3.x series of the [Python](http://python.org) language, there's a
[powerful,
unused](http://ceronman.com/2013/03/12/a-powerful-unused-feature-of-python-function-annotations/)
feature in the standard: [type
annotations](http://www.python.org/dev/peps/pep-3107/).
As they currently stand, type annotations don't seem to do anything except set
some attributes of the function to which they're applied; however, I've been
thinking of a possible use for them: compiling Python to C++.

I'm aware of the amazing [Cython](http://cython.org/) project, and
[various](http://www.swig.org/) [other](http://docs.python.org/2/extending/)
[options](http://cens.ioc.ee/projects/f2py2e/) when it comes to compiling code
for faster Python execution.
There's even [Shed Skin](https://code.google.com/p/shedskin/), a compiler that
takes much of the 2.x Python standard to C++ already.
Thus, this idea is nothing more than an excuse for me to learn more computer
science, specifically compilers (and lexers and parsers, oh my!).
However, besides Python 3.x's type annotations, there's one other feature none
of these options use:[^1]
[C++11](http://www.stroustrup.com/C++11FAQ.html), the new C++ standard.

# C++11
C++11 is quite interesting; Bjarne Stroustrup, the creator of C++, says

>Surprisingly, C++11 feels like a new language: The pieces just fit together
>better than they used to and I find a higher-level style of programming more
>natural than before and as efficient as ever.

There's a huge amount of things that have changed in the new standard, but some
of my favorites are [type
inference](http://en.wikipedia.org/wiki/C%2B%2B11#Type_inference), [lambda
functions](http://en.wikipedia.org/wiki/C%2B%2B11#Lambda_functions_and_expressions),
and [ranged-based for
loops](http://en.wikipedia.org/wiki/C%2B%2B11#Range-based_for_loop).  These new
features make C++ a much more pleasant language to work with, in my opinion; in
fact, I like it so much that I used it for the [code
portion](https://github.com/genos/e2c2) of my dissertation.

The above-mentioned type inference, coupled with [alternative function
syntax](http://en.wikipedia.org/wiki/C%2B%2B11#Alternative_function_syntax),
are what gave me the (probably silly) idea of trying to compile Python 3.x code
to C++11 code.

# Suggestive examples
Here are some slightly suggestive code examples that got me thinking.
The idea would be to somehow compile (perfectly valid) Python 3 code like this:
{% highlight python %}
def f(x:int, y:int) -> int:
    return 2 * x + y
{% endhighlight %}
into equivalent[^2] C++11 code:
{% highlight c++ %}
auto f(int x, int y) -> int {
    return 2 * x + y;
}
{% endhighlight %}
Here's a slightly more involved example: what if we could compile a (_very_
simple) Python class
{% highlight python %}
class R2Point(object):
    __slots__ = ('x', 'y')

    def __init__(self, x:int, y:int) -> None:
        self.x = x
        self.y = y

    def __add__(self, other:R2Point) -> R2Point:
        return R2Point(self.x + other.x, self.y + other.y)
{% endhighlight %}
into a C++11 struct
{% highlight c++ %}
struct R2Point {
    int x, y;
    auto operator+(R2Point other) -> R2Point {
        return R2Point{x + other.x, y + other.y};
    }
};
{% endhighlight %}
We can even handle Python lambdas[^3], to some extent.
This,
{% highlight python %}
f = lambda x: 2 * x
xs = xrange(10)
ys = []
for x in xs:
    ys.append(f(x))
print(sum(ys))
{% endhighlight %}
might become
{% highlight c++ %}
template <typename T> auto f = [](const T& x) { return 2 * x; }
auto xs = std::vector<int>(10); std::iota(xs.begin, xs.end, 0);
auto ys = std::vector<decltype(xs.front())>;
for (auto x: xs) {
    ys.push_back(f(x));
}
std::cout << std::accumulate(ys.begin(), ys.end(), 0) << std::endl;
{% endhighlight %}

Admittedly, these aren't very involved examples; the code is quite short and
simple, and they don't involve translating Python idioms like list
comprehensions into C++.
I still find them suggestive, though (at least, the first two).

# Something else
Like I said, these aren't terribly involved examples and don't demonstrate
changing idiomatic Python code into idiomatic C++11 code, really.
If the Python to C++ idea doesn't pan out, though, there's always the option of
something analogous to the relationship between
[CoffeeScript](http://coffeescript.org/) and
[JavaScript](http://en.wikipedia.org/wiki/JavaScript).
Since one of the [common
criticisms](http://gigamonkeys.wordpress.com/2009/10/16/coders-c-plus-plus/) of
C++ is that it is too complex, a simpler front end that compiles to a C++
back end might be interesting.
That being said, I think that starting fresh with idiomatic C++11 might be
enough of a simplification in some cases.

# In the long run
In the long run, I doubt this idea is really going to take off; Pythonistas
like Python for their reasons, and C++ coders like their language for their
own.
More than anything, any potential transpiler of Python to C++ I develop would
be a chance for me to learn more about how programming languages are
implemented and how compilation works.
I had a lot of fun reading [this](http://createyourproglang.com/) stellar book,
and might take its approach to this problem.
I just thought it was interesting how similar the syntax of the two languages
can look in their latest versions, and that got me wondering.
If I take this idea anywhere, I'll be sure to write about it.

[^1]: As far as my cursory research has shown; please feel free to correct me if I'm wrong!
[^2]: Well, largely equivalent; Python has arbitrarily large integers, which C++ can't match natively (even using `intmax_t`).
[^3]: Which are [weak](http://c2.com/cgi/wiki?PythonProblems)---some might say [broken](http://math.andrej.com/2009/04/09/pythons-lambda-is-broken/)---compared to C++11's (or Lisp's, Scheme's, Haskell's, etc.).
