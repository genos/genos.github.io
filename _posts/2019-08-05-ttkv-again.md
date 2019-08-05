---
title: "Time Traveling Key-Value Store, Again"
layout: post
---

# Introduction

In my [last post]({% post_url 2019-04-16-time-traveling-kv-store %}) I talked
about a time traveling key-value store implementation in some functional Scala.
After thinking about the problem more---[probably a little too
much](https://xkcd.com/356/)---I decided to try my hand at another
implementation, this time in Python again.
Rather than going the purely functional route, though, I wanted to focus on how
we represent the data internally.
Since we essentially have tabular data with three columns (timestamp, key, and
value), it made some sense to try and work with a table via the `sqlite3`
module in the [Python standard
library](https://docs.python.org/3/library/sqlite3.html).

# Implementation

First, some imports:

```python
from pickle import dumps, loads
from sqlite3 import Row, connect
from time import perf_counter
from typing import Any, List, Optional
```

We use the `pickle` module to marshal (almost) arbitrary keys and values to
binary blobs, `time` to give us a global clock as in my in-the-room
implementation from the interview, and `typing` to label our types.

Since I wanted to wrap all this up with a stable release of
[`nix`](https://nixos.org/nix/), I needed to use Python 3.6; this meant that
`time.perf_counter_ns()` wasn't available, so we make our own out of
`time.perf_counter()`:

```python
def perf_counter_ns() -> int:
    return int(perf_counter() * 1e9)
```

If one feels really strongly about Python 3.7, one could use the unstable
`nixpkgs` channel.

Next we create a `TTKV` class:

```python
class TTKV:

    def __init__(self) -> None:
        self.conn = connect(":memory:")
        self.conn.row_factory = Row
        self.cursor = self.conn.cursor()
        self.cursor.execute(
            "create table ttkv (timestamp integer, key blob, value blob)"
        )
        self.cursor.execute("create index timestamp_index on ttkv(timestamp)")
        self.cursor.execute("create index key_index on ttkv(key)")
        self.conn.commit()
```

Using this in-memory `sqlite` db, `put()` is an `INSERT` statement:

```python
    def put(self, key: Any, value: Any) -> None:
        self.cursor.execute(
            "insert into ttkv values (?, ?, ?)",
            (perf_counter_ns(), dumps(key), dumps(value)),
        )
```

While a `SELECT` statement takes care of `get()`:

```python
    def get(self, key: Any, timestamp: Optional[int] = None) -> Any:
        select = "select * from ttkv where key = ?"
        order = "order by timestamp desc"
        if timestamp is None:
            self.cursor.execute(f"{select} {order}", (dumps(key),))
        elif isinstance(timestamp, int):
            self.cursor.execute(
                f"{select} and timestamp <= ? {order}", (dumps(key), timestamp)
            )
        else:
            raise ValueError(timestamp)
        result = self.cursor.fetchone()
        if result is None:
            raise KeyError(key)
        return loads(result["value"])
```

To be more Pythonic, `get()` throws errors if given an unknown key or a
non-integral timestamp rather than going the Scala version's `Option` route.

Finally, the `times()` helper returns relevant timestamps like in the Scala
version: 

```python
    def times(self, key: Any = None) -> List[int]:
        select = "select timestamp from ttkv"
        order = "order by timestamp desc"
        if key is None:
            self.cursor.execute(f"{select} {order}")
        else:
            self.cursor.execute(f"{select} where key = ? {order}", (dumps(key),))
        return [row["timestamp"] for row in self.cursor.fetchall()]
```

# Testing

My
[`ttkv_spec.py`](https://github.com/genos/Programming/blob/master/workbench/ttkv_py/ttkv_spec.py)
looks similar to the previous
[`TTKVSpec.scala`](https://github.com/genos/Programming/blob/master/workbench/ttkv/src/test/scala/ttkv/TTKVSpec.scala),
using the wonderful [`hypothesis`
library](https://hypothesis.readthedocs.io/en/latest/) in place of `scalacheck`
for property testing.
I also lean on [`pytest`](https://docs.pytest.org/en/latest/) for running
the tests and for handling (deliberate) exceptions.
I'll copy just three tests here; interested readers should check the source on
GitHub for the rest.

```python
@given(STR)
def test_initially_empty(a: str) -> None:
    with raises(KeyError):
        TTKV().get(a)

@given(distinct_keys(), INT, INT)
def test_two_gets_different_keys(ab: Tuple[str, str], x: int, y: int) -> None:
    a, b = ab
    ttkv = TTKV()
    ttkv.put(a, x)
    ttkv.put(b, y)
    assert ttkv.get(a) == x
    assert ttkv.get(b) == y

@given(STR, INT, integers(min_value=0, max_value=SQLITE_MAX_INT))
def test_get_before_time(a: str, x: int, t: int) -> None:
    ttkv = TTKV()
    ttkv.put(a, x)
    t0 = ttkv.times(a)[0]
    with raises(KeyError):
        TTKV().get(a, t0 - t)
```


# Thoughts

I'm of two minds here.
Like the in-the-interview version I mentioned in my last post, this code relies
heavily on side-effects, using  its global nanosecond clock for timestamp
generation.
I liked how the functional Scala version explicitly isolated the effectful
`put()` computation.
However, I like how easily we map the `TTKV` API onto the `sqlite` table; the
`ORDER BY` clauses obviate the need to sort things, e.g.

It also seems that if the spec for `TTKV` changes at all (as software specs are
wont to do), we could easily modify this approach.
It also suggests using multiple data backends, wherein we might define an
abstract API and connect it to different specific data structures depending on
our needs.

As a happy accident of using the `pickle` module, this `TTKV` implementation
can use more types of things as keys than even the base `dict` class (e.g.
lists).
There are [restrictions on what can be
pickled](https://docs.python.org/3/library/pickle.html#what-can-be-pickled-and-unpickled),
of course, and this implementation doesn't handle any `PicklingError`s that
might occur if a use tries to save particular exotic keys or values.

I'm a _huge_ fan of
[`hypothesis`](https://hypothesis.readthedocs.io/en/latest/),
[`pytest`](https://docs.pytest.org/en/latest/), and
[`typing`](https://docs.python.org/3/library/typing.html) along with
[`mypy`](http://mypy-lang.org); I'd go so far as to say that I'd be reticent to
develop serious software in Python without them.
One (perhaps) interesting thing that happened in testing: though the `sqlite3`
module's documentation claims that one can make arbitrary Python integers into
native `sqlite` integer values, `hypothesis` quickly exposed a flaw in that
claim, running into `OverflowError`s when trying to save integral values too
large to fit into 64 bits.

# Appendix

Here's the `default.nix` file to specify the W O R L D:

```nix
let
  tgz = "https://github.com/NixOS/nixpkgs/archive/19.03.tar.gz";
  pkgs = import (fetchTarball tgz) { };
  py = pkgs.python36.withPackages (p: [ p.hypothesis p.pytest ]);
in pkgs.stdenv.mkDerivation {
  name = "ttkv";
  buildInputs = [ py ];
  shellHook = ''
    it () {
      pytest ttkv_spec.py
    }
  '';
}
```

To run the tests, I usually enter `nix-shell --pure --run it` on the command
line.
To run the tests yourself, poke around in the code, etc. feel free to copy the
whole `ttkv_py` directory from my [programming workbench on
GitHub.](https://github.com/genos/Programming/tree/master/workbench/ttkv_py)
