---
title: "Fourier Transform in Clojure, Reproducibly with Nix"
layout: post
---

# Combining previous work with new things

In a [previous post]({% post_url 2014-02-27-group-theory-and-statistics %}),
we explored an example from Persi Diaconis using the Fourier Transform to
change a collection of permutations into a table where the *(i, j)* entry is
the number of voters who ranked candidate *i* in place *j*; this
transformation consists of first mapping each permutation to its matrix
representation, then adding the matrices together.

There's more to explore here, however; since counts are just integers and
addition of integers is both associative and commutative, this matrix addition
is also commutative---that is, we can reorder and group the summands as much as
we like.
Thus, this problem is well suited to map-reduce-like approaches, since
associativity implies parallelism.
With commutativity, we can include a "shuffle" step, too.
This is what the `Clojure` library [`tesser`](https://github.com/aphyr/tesser)
provides.
Combined with [`core.matrix`](https://github.com/mikera/core.matrix) for
matrices, `tesser` allows us to

1. map the vote vectors to their permutation matrices
2. add them together

but, contrasted with core `Clojure`'s `map` and `reduce`, we can take advantage
of the mathematical structure to do all of step 1 in parallel, then all of
step 2 in parallel with arbitrary shuffling.

# `Clojure` Code

Here's `Clojure` code that, after simulating a collection of votes, calculates
the desired table in two different ways:

1. in serial, with the default `map` and `reduce`, and 
2. using `tesser` to take full advantage of associativity and commutativity.

It then checks that the two answers (imaginatively called `a` and `b`) are
equal, then prints out `a`.

```clojure
(require '[tesser.core :as t])
(require '[clojure.core.matrix :as m])

(let
  [k 7
   n 10000
   zero (m/zero-matrix k k)
   vecs (for [_ (range n)] (shuffle (range k)))
   a (->> vecs
          (map m/permutation-matrix)
          (reduce m/add zero))
   b (->> (t/map m/permutation-matrix)
          (t/reduce m/add zero)
          (t/tesser [vecs]))
   matrices-equal (if (= a b) "Yes" "No")]
  (println (str "\n\nAre a and b equal? " matrices-equal "\n"))
  (m/pm a))
```

Which, on one run, gave the following output:

```
Are a and b equal? Yes

[[1448.000 1419.000 1434.000 1505.000 1444.000 1389.000 1361.000]
 [1428.000 1429.000 1430.000 1404.000 1384.000 1475.000 1450.000]
 [1466.000 1471.000 1367.000 1458.000 1443.000 1408.000 1387.000]
 [1427.000 1369.000 1434.000 1429.000 1452.000 1434.000 1455.000]
 [1393.000 1388.000 1481.000 1444.000 1380.000 1439.000 1475.000]
 [1412.000 1429.000 1447.000 1389.000 1415.000 1486.000 1422.000]
 [1426.000 1495.000 1407.000 1371.000 1482.000 1369.000 1450.000]]
```

# `Nix` Expressions

Code demos are fun, but too often they fall victim to bit-rot and stop working.
To combat this, we can strive for reproducibility, an important goal of [data
science](https://www.coursera.org/learn/reproducible-research).
One tool that can help is `nix`.

[`Nix`](https://nixos.org/nix/) is a package manager, like `homebrew` for `OS X`,
that, through a novel approach, goes a _very_ long way towards reproducibility.
By specifying everything in a declarative, functional language, we can be sure
we're building the same thing twice.
With `nix`, we can specify the _exact_ versions of `tesser` and `core.matrix`
we used, along with how to build them.
Then, we can provide a function to reproducibly run the above example.
The `nix` expression to do so is rather long---though that's a small price to
pay for the assurance we get.

Here's the content of `default.nix`.
Save this in a file called `default.nix`, and save the above `Clojure` code in `diaconis.clj` in the same directory.
Then run `nix-shell --pure --run it` to see (eventually) the results.

```nix
with import <nixpkgs> {};
let
  buildLeinFromGitHub = { name, owner, repo, rev, sha256, cd }:
  stdenv.mkDerivation {
    name = name;
    src = fetchFromGitHub {
      owner = owner;
      repo = repo;
      rev = rev;
      sha256 = sha256;
    };
      buildInputs = [ leiningen ];
      buildPhase = ''
        # https://blog.jeaye.com/2017/07/30/nixos-revisited/
        export LEIN_HOME=$PWD/.lein
        mkdir -p $LEIN_HOME
        echo "{:user {:local-repo \"$LEIN_HOME\"}}" > $LEIN_HOME/profiles.clj
        cd ${cd}
        LEIN_SNAPSHOTS_IN_RELEASE=1 ${leiningen}/bin/lein uberjar
      '';
      installPhase = ''
        cp target/${repo}*-standalone.jar $out
      '';
  };
  tesser = buildLeinFromGitHub {
    name = "tesser";
    owner = "aphyr";
    repo = "tesser";
    rev = "b7b67dfaf25f1764c70c90dc6681dd333d24d6a4";
    sha256 = "1vma6ram4qs7llnfn84d4xvpn0q7kmzmkmsjpnkpqnryff9w2gvr";
    cd = "core";
  };
  matrix = buildLeinFromGitHub {
    name = "matrix";
    owner = "mikera";
    repo = "core.matrix";
    rev = "f864c29d4e85d35de018295a87a295fc3df632a6";
    sha256 = "1dywj2av5rwnv7qhh09lpx9c2kx7wvgllwyssvyr75cb6fa6smvg";
    cd = ".";
    };
in
  stdenv.mkDerivation {
    name = "diaconis.clj";
    src = ./.;
    buildInputs = [ clojure jdk ];
    shellHook = ''
      it () {
        ${jdk}/bin/java -cp ${tesser}:${matrix}:${clojure}/share/java/clojure.jar clojure.main ${./diaconis.clj}
      }
    '';
  }
```
