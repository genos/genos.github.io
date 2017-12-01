---
title: "Unusual tools in Data Science"
layout: post
---

# Isn't *all* science "Data Science?"
I'm an applied research mathematician in my day job, which on a day-to-day
basis puts me somewhere between mathematician, software developer, machine
learning practitioner, and statistician.
In other words, I'm probably what's called a *Data Scientist* nowadays.
See the image below for how some people feel about that term;[^1] I'm a bit
ambivalent about it myself.

It seems to be a bit trendy and not have much of a solid definition.
Some people even think [the title should be killed
off](http://blogs.wsj.com/cio/2014/04/30/its-already-time-to-kill-the-data-scientist-title/)
entirely.
That being said,

1. I'm mostly OK with applying a trendy label to myself
2. I don't have a better term for the "person who uses software to do
   interesting things with data" right now

so I'll stick with it for the time being.

![Data
Scientist](https://d262ilb51hltx0.cloudfront.net/max/800/1*194NFTNMbRww0eTlhHfqlQ.png)

# A series idea
In my time working in "data science," I've noticed that there are a number of
interesting tools out there that tend to go unnoticed.
Perhaps it's because people that fit under the data science umbrella have wide
and varied backgrounds, but there seems to be a lack of agreement of best
practices---or a lack of following best practices even if folks agree.
Moreover, there doesn't seem to be much awareness about the wide array of new,
interesting, or just plain different (computational) tools that are out
there.[^2]

There are a number of tools out there that can solve certain problems much more
cleanly, elegantly, or efficiently than the [ones](http://www.r-project.org/)
people
[usually](https://www.kaggle.com/wiki/GettingStartedWithPythonForDataScience)
reach for.
Though of course the usual tools have a lot of merit---[this
book](http://shop.oreilly.com/product/0636920023784.do) is particularly
nice---there's definitely room for more tools in the data science tool belt.

# Post the first
The first tool I'll mention is Haskell for parsing various types of logs and
files.
Several times I've ended up having to parse a log or other type of file that,
while it follows a specific format, isn't one of the usual suspects like SQL or
CSV.
Rather than reaching for regexes,[^3] I contend that this is exactly the type
of problem that Haskell---with
[Parsec](http://hackage.haskell.org/package/parsec) or (even better)
[Attoparsec](https://hackage.haskell.org/package/attoparsec)---can easily
handle.

I won't make this post into an introduction to parsing with
[Attoparsec](https://github.com/bos/attoparsec); there are
already
[great](https://www.fpcomplete.com/school/starting-with-haskell/libraries-and-frameworks/text-manipulation/attoparsec)
intros out there that do a better job than I could.
I'll just echo the
[experiences](http://newartisans.com/2012/08/parsing-with-haskell-and-attoparsec/)
of
[others](https://variadic.me/posts/2012-02-25-adventures-in-parsec-attoparsec.html)
out there and let you know that parsing ad hoc (or just new) file formats with
Attoparsec leads to solutions that are

- easier to read that regexes
- easier to reason about than regexes
- easier to come back to and work on/extend later than regexes
- ... and typically *very* fast

Moral of the story: if you're doing data science, don't be afraid to use
different tools than you're used to; they just might lead to better solutions.

[^1]: The post from which I <s>stole</s> borrowed that picture is also [a pretty good read.](http://slantedwindows.com/why-you-already-are-a-data-scientist-its-just-may-not-be-your-job-title/)
[^2]: Or perhaps that's just at my workplace.
[^3]: Which I've also done because Haskell wasn't an option (the people I was working with **only** liked *Perl*), and it lead to some horrific circumstances similar to [this](http://stackoverflow.com/questions/1732348/regex-match-open-tags-except-xhtml-self-contained-tags/1732454#1732454).
