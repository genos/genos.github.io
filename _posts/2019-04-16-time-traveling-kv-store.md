---
title: "A Time Traveling Key-Value Store"
layout: post
---

# Never Give Up

Once upon a time I interviewed for an ML/DS/software engineering position.
I didn't get the gig (as the inimitable Time Hopper says, [never give
up](https://twitter.com/tdhopper/status/711948018224799746)), but came away
with a fun programming exercise.
In the room, I put together some reasonable Python to solve and test it, but I
wanted to try my hand at using
[`cats-effect`](https://typelevel.org/cats-effect/) to work through this in
some functional Scala.

# What is Your Problem and Why is it Interesting?

Your mission, should you choose to accept it, is to implement (and test) a data
structure that looks a bit like a dictionary/map/hash/associative
array/whatever your language calls it, but with a twist.
In pseudo-Scala, your *Time Traveling Key-Value Store* (TTKV from now on)
should have two methods that look a bit like the following (note that we change
this a little in our implementation for functional programming goodness):

```scala
def put(key: K, value: V): Unit
def get(key: K, time: Option[Long] = None): Option[V]
```

Calling `put` should store the `(key, value)` pair in your TTKV.
Calling `get`, however, has some interesting behavior.
If you don't supply an optional `time`, it returns the latest `value` you stored
with the `key`.
If you do supply a `time`, TTKV returns the `value` it had for `key` at that
`time`.
For instance, if you first call `TTKV.put("a", 1)` at time 3, then later at
time 7 you call `TTKV.put("a", 2)`, then

- `TTKV.get("a")` should be `Some(2)`,
- `TTKV.get("a", Some(6))` should be 1, since 6 is after 3 but before 7,
- `TTKV.get("a", Some(n))` should be `Some(2)` for any `n ≥ 7`
- `TTKV.get("a", Some(n))` should be `None` for any `n < 3`
- `TTKV.get(x, y)` should be `None` for any other `x` and `y`.

My Python in-the-room implementation relied on side-effects, reading a global
[nanosecond
clock.](https://docs.python.org/3.7/library/time.html#time.process_time_ns)
This was less than ideal, and though it met the remit, it wasn't easily
testable---which I suspect was the point of the exercise.

By its nature, `put` involves an effect: reading a time.
To do things functionally, I'll turn to `cats-effect` and explicitly make note
of this effectful computation.
This involves a slight change to TTKV's API; while `get` stays the same, we'll
change `put` to the following:

```scala
def put[F[_]](key: K, value: V)
             (implicit sync: Sync[F], clock: Clock[F]): F[TTKV[K, V]]
```

First, rather than mutating `TTKV` in place, `put` returns a new one with the
`(key, value)` pair added for a specific time.
Second, we're explicitly noting that this work takes place inside a `Clock`
effect, from which we grab the current time.
I like how we can still `get` the same way as before, since querying a TTKV
shouldn't have to concern itself with effects, but this design specifically
highlights that `put` is different.

# Implementation

Here's my implementation in its entirety (again, head to
[GitHub](https://github.com/genos/Programming/tree/master/workbench/ttkv) to
see it in action):

```scala
package ttkv

import cats.data.{NonEmptyList => NEL}
import cats.effect.{Clock, Sync}
import cats.implicits._
import scala.concurrent.duration.NANOSECONDS
import scala.collection.immutable.LongMap

case class TTKV[K, V](inner: Map[K, LongMap[V]]) extends AnyVal {
  def put[F[_]](key: K, value: V)
               (implicit sync: Sync[F], clock: Clock[F]): F[TTKV[K, V]] =
    clock.monotonic(NANOSECONDS).map { t =>
      val m = inner.getOrElse(key, LongMap.empty) + (t -> value)
      TTKV(inner + (key -> m))
    }

  def get(key: K, time: Option[Long] = None): Option[V] =
    for {
      m <- inner.get(key)
      keys <- NEL.fromList {
        m.keys.toList.filter(_ <= time.getOrElse(Long.MaxValue))
      }
      t0 <- keys.maximumOption
      value <- m.get(t0)
    } yield value

  def times(key: K): Option[NEL[Long]] =
    inner.get(key).flatMap(m => NEL.fromList(m.keys.toList.sorted))

  def times: Option[NEL[Long]] =
    for {
      ks <- NEL.fromList(inner.keys.toList)
      ts <- ks.traverse(times)
    } yield ts.flatten.sorted
}

object TTKV {
  def empty[K, V]: TTKV[K, V] = TTKV(Map.empty)
}
```

# Will it Blend?

In the room, I wrote some simple unit tests for my Python implementation.
For this functional version, I put together some property-based tests with
[`ScalaCheck`.](http://www.scalacheck.org)
You can find the entirety on
[GitHub](https://github.com/genos/Programming/tree/master/workbench/ttkv); I'll
copy some highlights here.

First off, calling `get` on an empty TTKV should return `None`:

```scala
property("initially empty") = forAll {
  (a: Char) =>
    TTKV.empty.get(a) == None
}
```

Similarly, calling `get` immediately after a `put` should return the same thing
you `put`:

```scala
property("single get") = forAll {
  (a: Char, x: Int) =>
    TTKV.empty.put(a, x).map(_.get(a) == Some(x)).unsafeRunSync
}
```

More importantly, calling `get` with any time prior to when you `put` a thing
should return `None`:

```scala
property("get before time") = forAll {
  (a: Char, x: Int, t: Long) =>
    (t > 0) ==>
      TTKV.empty
        .put(a, x)
        .map { m =>
          val t0 = m.times(a).get.head
          m.get(a, Some(t0 - t)) == None
        }.unsafeRunSync
}
```

And for two `puts` with the same key, calling `get` with any time _between_ the
two times should return the value from the _first_ `put`:

```scala
property("middle get") = forAll {
  (a: Char, x: Int, y: Int, d: Double) =>
    ((0 <= d) && (d < 1)) ==>
      TTKV.empty
        .put(a, x)
        .flatMap(_.put(a, y))
        .map { m =>
          val ts = m.times(a).get
          val t0 = ts.head
          val t1 = ts.last
          val δ = (t1 - t0) * d
          m.get(a, Some(t0 + δ.toLong)) == Some(x)
        }
        .unsafeRunSync
}
```

# Appendix

Here's the `default.nix` file to specify the W O R L D:

```nix
let
  pkgs = import (
    fetchTarball https://github.com/NixOS/nixpkgs/archive/19.03.tar.gz
  ) {};
in
  pkgs.stdenv.mkDerivation rec {
    name = "ttkv";
    buildInputs = [ pkgs.sbt ];
    shellHook = ''
      it () {
        ${pkgs.sbt}/bin/sbt test
      }
    '';
  }
```

The `build.sbt` is similarly minimal:

```scala
lazy val root = (project in file("."))
  .settings(
    name := "ttkv",
    libraryDependencies ++= Seq(
      "org.typelevel" %% "cats-effect" % "1.2.0",
      "org.scalacheck" %% "scalacheck" % "1.14.0" % "test"
    ),
    scalacOptions ++= Seq(
      "-feature",
      "-deprecation",
      "-unchecked",
      "-language:postfixOps",
      "-language:higherKinds",
      "-Ypartial-unification"
    )
  )
```

To run the tests, poke around in the code, etc. feel free to copy the whole
`ttkv` directory from my [programming workbench on
GitHub.](https://github.com/genos/Programming/tree/master/workbench/ttkv)
