---
title: "Tiny REST API Demo in Python"
layout: post
---

# Introduction

The other day, someone asked me about the difference between "an API and a regular webpage."
After understanding more about the context of their question, I tried to come up with a decent explanation about the differences between a server sending HTML pages and one handling REST API requests.
As [a thousand words leave not the same deep impression as does a single deed](https://en.wikipedia.org/wiki/A_picture_is_worth_a_thousand_words#History), I thought I'd put together a tiny demonstration that serves HTML to a web browser and JSON over a REST endpoint.
My demo uses the [Flask](https://flask.palletsprojects.com/) microframework to handle the details; you can find a copy [here](https://github.com/genos/tiny_api_demo), though we'll go through some of it below.

# How smol?

Before we get into the specifics, here's how tiny the demo is: the whole thing, including a `flake.nix` file to set the stage, is under 115 lines.

```
~/github/tiny_api_demo ∃ tokei
===============================================================================
 Language            Files        Lines         Code     Comments       Blanks
===============================================================================
 CSS                     1           10           10            0            0
 HTML                    3           41           41            0            0
 Markdown                1           10            0            6            4
 Nix                     1           25           23            0            2
 Python                  2           54           40            2           12
===============================================================================
 Total                   8          140          114            8           18
===============================================================================
```

This whole thing cuts so many corners it might as well be a sphere.
It's _certainly_ not a production-ready example by _any_ stretch of the imagination; it's only enough for demonstration purposes.

# Preliminaries

First, we need some data.
Pretend we're creating the _School Reporter 5000™_, wherein we have student information that consists of a collection of `(timestamp, person, activity)` tuples.
In `data.py`, we generate some fake data.
Our application code will interface with this data via `NAMES` and `SELECT`; pretend this stands in for a proper database.

```
from datetime import datetime, timedelta
from itertools import accumulate
from operator import add
import random

random.seed(1729)

NAMES = ["Alice", "Bob", "Carol", "Dave"]
_ACTIVITIES = ["Reading", "Writing", "Arithmetic"]
_DATA = [
    {
        "timestamp": ts.time().isoformat(timespec="minutes"),
        "person": random.choice(NAMES),
        "activity": random.choice(_ACTIVITIES),
    }
    for ts in accumulate(
        (timedelta(minutes=random.randrange(17)) for _ in range(42)),
        func=add,
        initial=datetime.today().replace(hour=9),
    )
]
SELECT = {name: [x for x in _DATA if x["person"] == name] for name in NAMES}
```

# App

The _School Reporter 5000™_ has three routes, all described in `app.py`.
We serve HTML for the homepage at `/index.html` and individual person pages at `/person/<name>` for each of the names in `NAMES`.
We also serve JSON in response to GET requests via the `/api/<name>` route.
Thanks to [Flask](https://flask.palletsprojects.com/), we even have rudimentary error handling.

```
from flask import Flask, jsonify, render_template
from data import NAMES, SELECT

app = Flask(__name__)

@app.route("/")
def home():
    return render_template("index.html", names=NAMES)

@app.route("/person/<string:name>")
def person(name):
    try:
        return render_template("person.html", name=name, data=SELECT[name], error=False)
    except KeyError:
        return render_template("person.html", name=name, error=True), 400

@app.route("/api/<string:name>")
def api(name):
    try:
        return jsonify(SELECT[name])
    except KeyError:
        return {"error": f"Unknown person {name}"}, 400
```

With the server running, we can visit the home page:

<img src="{{ "/public/tiny_api_demo_index.png" | absolute_url }}" width="42" alt="Homepage">

Or report pages for any specific student:

<img src="{{ "/public/tiny_api_demo_alice.png" | absolute_url }}" width="42" alt="Alice's page">

However, we can also get JSON data for individual students by pinging the REST endpoint; using, e.g. [HTTPie](https://httpie.org) to query the endpoint via `http --body :5000/api/Dave`, we get

<img src="{{ "/public/tiny_api_demo_dave.png" | absolute_url}} " width="75" alt="Dave's JSON">


# Everything Else

To round out the example, we also have [some minimal CSS](https://github.com/genos/tiny_api_demo/blob/main/static/styles.css) and [two other HTML templates](https://github.com/genos/tiny_api_demo/blob/main/templates).
Here's the `flake.nix` to specify the W O R L D:

```nix
{
  description = "Tiny API Demo";

  inputs = {
    flake-utils.url = "github:numtide/flake-utils";
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-22.11";
  };

  outputs = {
    self,
    flake-utils,
    nixpkgs,
  }:
    flake-utils.lib.eachDefaultSystem (system: let
      pkgs = import nixpkgs {inherit system;};
      py = pkgs.python310.withPackages (p: [p.flask]);
    in {
      packages.default = pkgs.writeShellApplication {
        name = "tiny_api_demo";
        text = ''
          FLASK_APP=app.py ${py}/bin/python -m flask run
        '';
      };
    });
}
```

From the base directory, one can run `nix run` to set up the environment and kick off the server.
