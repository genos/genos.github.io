---
title: "Silly Shenanigans"
layout: post
---

# Introduction

In an effort to both post more than once a year and clean out some silly things
from my `~/tmp` directory, here's a quick post about two programming
shenanigans.
Perhaps unsurprisingly due to its history of jocularity, both originate with
Perl.

# Bleach

[Damian Conway](https://en.wikipedia.org/wiki/Damian_Conway) is a truly prolific
member of the Perl community.
One of Conway's less serious contributions is the
[`Acme::Bleach`](https://metacpan.org/pod/Acme::Bleach) module, famous for
being the [first `Acme` module and the first source filter joke
module.](https://www.perlmonks.org/?node_id=967004)
`Acme::Bleach` takes programs that look like this:

```perl
use Acme::Bleach;
print "Hello world";
```

and turns them into

```perl
use Acme::Bleach;
 	 	 	 	 	 	 	 			   	   
 		 		 	 
 	 		  		
	 		 	 	 
 		      
	  	   		
       	 
 			 			 
 			 	   
   			   
		 		  	 
	        
			  	  	
		 	  	 	
	  			 		
   	 			 
     	   
	   	    
 	  	 	 	
  		   		
 		   		 
		 				 	
	      	 
 			 			 
				 		  
	  			   
		 		   	
  		  	  
 	  		 		
	   	 	  
  
```

No, that's not an error.
Through the magic of Perl's [source
filters](https://perldoc.perl.org/perlfilter.html), the above program whose
source consists of nothing but one import statement and a bunch of whitespace
_still prints "Hello world"!_

After digging through the source, I decided to try my hand at recreating it in
Python:

```Python
#!/usr/bin/env python3
"""For really clean programs"""

from functools import reduce
from pathlib import Path
from subprocess import run
from sys import argv

ZERO = ord(" ")
ONE = ord("\t")

def zero_one(x: int) -> bytes:
    """Convert a byte to a series of `ZERO` and `ONE`"""
    return bytes([ZERO, ONE][(x >> i) & 1] for i in range(7, -1, -1))

def un_zero_one(xs: bytes) -> int:
    """Undo `zero_one`"""
    return reduce(lambda i, x: (i << 1) | int(x == ONE), xs, 0)

def encode(xs: bytes) -> bytes:
    """Use `zero_one` to encode"""
    return b"".join(zero_one(x) for x in xs)

def decode(xs: bytes) -> bytes:
    """Use `un_zero_one` to decode"""
    return bytes(un_zero_one(xs[i : i + 8]) for i in range(0, len(xs), 8))

if __name__ == "__main__":
    if len(argv) != 2:
        exit("I need a file to bleach")
    else:
        XS = Path(argv[1]).read_bytes()
        if any(x not in {ZERO, ONE} for x in XS):
            print(encode(XS).decode("ascii"), end="", flush=True)
        else:
            run(["python3", "-"], input=decode(XS))
```

Since Python doesn't have Perl's source filters, we can't do everything that
`Acme::Bleach` does, but `bleach.py` uses the same main trick: source code
consists of bytes, as far as computers care, so we can encode and decode them
however we want—including using spaces for zeros and tabs for ones.
If we have a file called `simple.py` consisting of `print("hello world")`,
we can say `./bleach.py simple.py > simple_bleached.py`.
Then running `./bleach.py simple_bleached.py` will result in the interpreter
printing `"hello world"`.

# D&D

I recently came across a [fun
post](https://aearnus.github.io/2019/08/07/d-d-rolls-in-perl-6) that describes
implementing basic dice notation from Dungeons & Dragons in Perl 6.
It's a fun read, so you should definitely check it out, but its basic
implementation looks like this:

```perl
sub infix:<d>(Int $n, Int $max) { (1..$max).pick xx $n }
```

So one can enter `say (3 d 20);` and see something like `(17 1 12)` printed.
Haskell has nice support for infix-ing any function by wrapping it in
backticks, so I came up with the following `dnd.hs`:

```haskell
#!/usr/bin/env stack
-- stack --install-ghc runghc --package random --resolver lts-14.0
module Main where

import Data.Foldable (traverse_)
import System.Random (getStdGen, randomRs)

d :: Word -> Word -> IO [Word]
d n size = take (fromIntegral n) . randomRs (1, size) <$> getStdGen

main :: IO ()
main = traverse_ (print =<<) [3 `d` 8, 2 `d` 20, 1 `d` 6]
```

Running it via `./dnd.hs`[^1], I got

```
[6,6,3]
[10,10]
[2]
```

Though of course different runs will probably output different results.

[^1]: Assuming of course that you've got [`stack`](https://en.wikipedia.org/wiki/Stack_(Haskell)) installed.
