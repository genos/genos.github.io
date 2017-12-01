---
title: "ScalaCheck and the Unkillable Trash"
layout: post
---

# Well that's weird
I ran into an interesting problem in a recent
[`Scala`](http://www.scala-lang.org/) project.
Let's set the scene &hellip;

# Quickly Checking in Scala
In the project in question, I reached for [a library](http://argonaut.io/) to
do a fair amount of JSON parsing.
I hadn't used the library before, so I ended up writing some
[`ScalaCheck`](https://www.scalacheck.org/) tests for the convenience/wrapper
functions I made.

One test involved writing data to a temporary file, reading and parsing that
information, then comparing the parsed results to the original data.
If all went well, the parsed and original data should match perfectly.
Either way, there was clean up code at the end of the test to delete the
temporary file.
My mistake was in letting `ScalaCheck` generate strings of arbitrary characters
for the temporary file's name, rather than restricting those filenames to
printable or alphanumeric characters only.

# Obscure OS X obstacles
It turns out that the latest version of Mac OS X can't delete&mdash;or
otherwise get a handle on, via GUI or CLI&mdash;files with [certain characters
in the
name](http://apple.stackexchange.com/questions/224505/is-it-impossible-to-delete-move-a-file-named-on-mac).
So, when `ScalaCheck` created such a temporary file, my clean up code was
unable to `rm` it.
I tried *everything* the [Quantifind Dev team](http://quantifind.com/) and I
could think of, to no avail.
I now have, tucked away in a directory called `unkillable_trash`, a file that
nothing short of reformatting my hard drive will remove.

# Update (2017-04-13)
Since the time of this post, a botched update necessitated reinstalling the OS
on my work computer; the `unkillable_trash` is, sadly, now gone.
