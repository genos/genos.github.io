---
title: "Richard Stanley, Catalania, and Haskell"
layout: post
use_katex: true
---

# Richard Stanley, Captain of Counting
I recently received an email about a [math
conference](http://math.mit.edu/stanley70/Site/Home.html) in honor of Richard
Stanley's 70th birthday.
For those of you who don't know, Stanley is an expert in combinatorics and
author of the two volume magnum opus [Enumerative
Combinatorics](http://www-math.mit.edu/~rstan/ec/) which (among a bunch of
other very interesting math) champions the idea of the [combinatorial
proof](http://en.wikipedia.org/wiki/Combinatorial_proof)---a way of proving a
combinatorial identity by showing that both sides of the equation are in
different fact ways of counting the same thing.

Stanley's Enumerative Combinatorics also goes into great detail about the
[Catalan Numbers](http://mathworld.wolfram.com/CatalanNumber.html), a sequence
of numbers that arises in tree enumeration problems.
In fact, Stanley's interest in these numbers---which he dubs ''Catalan
disease'' or ''Catalania''---goes so far that he offers 66 combinatorial
interpretations of them in EC vol. 2 and dozens more in an addendum published
[online](http://www-math.mit.edu/~rstan/ec/catadd.pdf).
See the [OEIS
page](http://oeis.org/wiki/Combinatorial_interpretations_of_Catalan_numbers)
for more.

# Catalania, meet Haskell
In the interest of learning more Haskell, I decided to take some inspiration
and play with Catalan numbers in that decidedly mathematical programming
language.
To start off with, I have some helpful imports and auxiliary functions:
{% highlight haskell %}
import Test.QuickCheck (quickCheck)

-- helpers
binom :: Integer -> Integer -> Integer
binom n k = product [k + 1..n] `div` product [1..n - k]

factorial :: Integer -> Integer
factorial = product . enumFromTo 2

pairs :: [a] -> [(a, a)]
pairs xs = [(x, y) | x <- xs, y <- xs]

{% endhighlight %}
The [QuickCheck](http://www.cse.chalmers.se/~rjmh/QuickCheck/) import will be
important later.

Next, with the help of [Rosetta
Code](http://rosettacode.org/wiki/Catalan_numbers#Haskell) and Wikipedia, I
have the following Catalan Number definitions:
{% highlight haskell %}
-- Catalan definitions
cat0 :: [Integer]
cat0 = map (\n -> product [n + 2..2 * n] `div` product [2..n]) [0..]

cat1 :: [Integer]
cat1 = 1 : map c [1..]
  where c n = sum $ zipWith (*) (reverse (take n cat1)) cat1

cat2 :: [Integer]
cat2 = scanl (\c n -> c * 2 * (2 * n - 1) `div` (n + 1)) 1 [1..]

cat3 :: [Integer]
cat3 = map c [0..] where
  c n | n <= 0    = 1
      | otherwise = binom (2 * n) n - binom (2 * n) (n + 1)

cat4 :: [Integer]
cat4 = map c [0..]
  where c n = sum (map (\k -> binom n k ^2) [0..n]) `div` (n + 1)

cat5 :: [Integer]
cat5 = map (\n -> binom (2 * n) n `div` (n + 1)) [0..]

cat6 :: [Integer]
cat6 = map c [0..]
  where c n = factorial (2 * n) `div` (factorial (n + 1) * factorial n)
{% endhighlight %}

Finally, we'd like to show (or at least, verify for a certain number of test
cases) that these varying definitions all describe the same thing; to do that,
I decided to use a bit of QuickCheck's magic to test various items from these
lists for equality:
{% highlight haskell %}
-- Check that these look the same
prop_equal :: Int -> Bool
prop_equal n = all p $ pairs cats
  where
    cats = [cat0, cat1, cat2, cat3, cat4, cat5, cat6]
    p (xs, ys) = xs !! n' == ys !! n'
    n' = n `mod` 1000

main :: IO ()
main = quickCheck prop_equal
{% endhighlight %}
This isn't perhaps the most elegant check---note that I restrict the integers
tested to \\( \lbrace 0, 1, \ldots, 999 \rbrace
              = \phantom{ }^\mathbb{Z} /_{1000 \mathbb{Z}} \\) to make sure
that indices are nonnegative and we don't overtax my poor laptop's CPU.
Of course, I'm open to criticism from any Haskell/QuickCheck gurus who'd like
to help me tighten up my code.
Feel free to [download the
code](https://github.com/genos/Programming/blob/main/workbench/catalan.hs).

# To Be Continued
That was fun; programming in Haskell almost always is, for me, and I'm by no
means an expert.
However, it leaves a little to be desired.
The program above at best demonstrates that the various infinite lists agree at
several (pseudo-)randomly chosen indices.
I suppose that taking this to the next level would involve some sort of
automatic theorem proving
assistant---[Agda](http://wiki.portal.chalmers.se/agda/pmwiki.php) comes to
mind, due to its relationship with Haskell.
That being said, using something like Agda will at best give me _algebraic_
proofs that \\( cat_i = cat_j \\) by exploiting various mathematical
properties of the definitions of two lists.
I highly doubt that it would be able to produce a _combinatorial_ proof that
two descriptions of the Catalan numbers are the same, though I'd be very
happy to be proven wrong.
