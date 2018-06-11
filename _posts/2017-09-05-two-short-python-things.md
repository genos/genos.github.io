---
title: "Two Short Python Things"
layout: post
---

# Sorry

Apologies, as ever, for taking _so_ incredibly long between posts.
That said if the inimitable Roger Peng can get away with letting some time
lapse while working on a project, perhaps you can forgive me?

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Sometimes it takes a while for a project to get going. <a href="https://t.co/gQoKSyetWr">pic.twitter.com/gQoKSyetWr</a></p>&mdash; Roger D. Peng (@rdpeng) <a href="https://twitter.com/rdpeng/status/900699027108282368">August 24, 2017</a></blockquote> <script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

# Back to Python

After a long time in the Scala wilderness, I've recently had a chance to write
some Python again for scripting purposes at work.
I had a bit of freedom with regards to process, so I explicitly chose Python
3.6.
I'm here to tell you that Python 3.6 is really nice and to exhibit two things
that made my life much better.

# What's up, `__doc__`?

Especially for scripts, I've gotten into the habit of putting all of the actual
work and logic of my Python program in functions (usually a `main` function
calling other steps/etc.).
I then have an `argparse.ArgumentParser` at the bottom (wrapped in an `if
__name__ == '__main__'` [scripting
statement](https://docs.python.org/3/library/__main__.html)) that parses
command line arguments and passes them to the `main` function, like so:

```python
if __name__ == '__main__':
    P = ArgumentParser()
    P.add_argument(…)
    …
    main(P.parse_args())
```

Given this setup, one can pass the `-h` or `--help` flag to the script and
receive some information about the different arguments you've added to the
parser.
However, it's good form to add a `description` to your `ArgumentParser` that
explains the purpose and usage of the script.

If you're following best practices (e.g. by trying to appease
[`pylint`](https://www.pylint.org)), you've already added a docstring to the
top of your file that explains its purpose and usage.
It can be a pain (or at least problematic) to keep this docstring and the
parser's `description` in sync.

I recently stumbled across a nice solution to this problem: only write the
docstring, and then use the [`__doc__` magic
variable](https://stackoverflow.com/questions/33066383/print-doc-in-python-3-script) as your `description` like so:

```python
if __name__ == '__main__':
    P = ArgumentParser(description=__doc__)
```

This approach does limit the length and expressiveness you can fit into the
docstring/description due to the constraints of printing a help message, but
I haven't found it to be a problem since my script docstrings tend to be
exactly what I'd want to print a help message anyway.

# `typing.NamedTuple` is my new favorite

Even more mindbending is the new (in Python 3.5) [`typing
module`](https://docs.python.org/3/library/typing.html).
Coupled with the [`mypy` static checker](http://mypy.readthedocs.io) and [type
hints](https://stackoverflow.com/questions/32557920/what-are-type-hints-in-python-3-5),
this allows you to write Python with static types and check them _prior_ to
running code in production.
For instance, the `main` function for my scripts that once looked like this:
```python
def main(args):
    """Performs the main work
    
    Parameters
    ----------
    args: Namespace
        Arguments passed on the command line, specifying what to do

    Returns
    -------
    None
    """
    # stuff and things
    …
```
now looks like this:
```python
def main(args: Namespace) -> None:
    """Performs the main work
    
    Parameters
    ----------
    args: Arguments passed on the command line, specifying what to do
    """
    # stuff and things
    …
```

Some folks might cry that this doesn't look like Python anymore, but to them I
counter
- [Python Enhancement Proposal 484](https://www.python.org/dev/peps/pep-0484/)
  already accepted type hints into the language, and
- if you're [writing suitably descriptive
docstrings](http://sphinxcontrib-napoleon.readthedocs.io/) for your functions,
you're _already_ writing the input and output types anyway; why not statically
check them, too?

If I haven't lost you with the type hinting above, I have something else to
show you: [the new `NamedTuple`
class.](https://docs.python.org/3/library/typing.html#typing.NamedTuple)
When writing Python classes, I often get annoyed with having to specify the
typical initialization code one finds in the `__init__` method, like

```python
class MyThing:

    def __init__(self, a, b, c, d):
        self.a = a  # this is an integer
        self.b = b  # this is a float
        self.c = c  # this is a string
        self.d = d  # this is a dictionary

    … # other methods here
```

Unless I _need_ some fancy initialization logic, I'd prefer to have this
attribute setup taken care of by default; enter `NamedTuple`:

```python
class MyThing(NamedTuple):
    a: int
    b: float
    c: str
    d: dict

    … # other methods here
```

Besides being shorter ("brevity facilitates reasoning," as [one of my favorite
papers](http://www.jsoftware.com/papers/tot.htm) explains), this version
_explicitly_ lists the attribute types in a way checkable via `mypy`.
For instance, `x = MyThing(1, 2, 'three', {'four': 4})` is just fine, but
if you try to sneak `y = MyThing(1.0, 2, 'three', {'four': 4})` past `mypy`
it'll complain
```
error: Argument 1 to "MyThing" has incompatible type "float"; expected "int"
```
Perhaps my brain has rotted away after relying on
[statically](https://haskell-lang.org) [typed](https://www.scala-lang.org)
[languages](https://www.rust-lang.org), but I'm a fan.

