---
title: "Lisp Adventures #2 - One, Two, Three... Infinity"
description: "SOMETHING HERE"
date: 2023-03-29T08:58:51+02:00
tags:
 - lisp
 - racket
 - functional programming
 - algorithms
---

Imagine you want to count the sum of the numbers from 1 to 1,000,000,000 (one billion). If you come from an imperative programming background, you might be already thinking on a loop and what type integer to hold that value. You might come up with something [similar to this](https://godbolt.org/z/Gos5Tvenn) if you're writing C:

```c
uint64_t sum = 0; 
for (int i = 1; i <= 1000000000; i++)  
    sum += i;  
printf("%lu\n", sum);
```

Good, it runs quite fast and we get the solution. But let's try to write a solution for that problem in a functional style Python:

```python
sum(range(1,1000000001))
```

It's more elegant and simpler. I like it. But I'm not satisfied and I want to do it in Racket:

```scheme
(apply + (range 1000000001))
```

Since Racket doesn't have a `sum` function, I'll use `apply` and pass the function `+` to it. But when I try to run, I get this error:

![Memory Error in DrRacket executing the naive approach](images/racket-apply-memory-error.jpg)

If I run that in a file invoking directly the `racket file.rkt`, I get an even more obscure message: `Killed`. But that makes sense, because the `apply` function basically gets every element in the list and apply as parameters to pass to the function specified. Example:

```scheme
(apply + (range 10))
```

Will expand to:

```scheme
(+ 0 1 2 3 4 5 6 7 8 9)
```

And considering that Racket uses big numbers as the default numeric type, I can imagine that it'll take a lot of memory. But that's unecessary, let's try a different way, let's fold that list:

```scheme
(foldl + 0 (range 1000000001))
```

Oh no. Same memory problem. But expected again, after all the `range` function creates a list with the numbers within that range, then we have the exact same problem as before. Which means that it's the `foldl` that is breaking our program, but the `range` itself. But what if we could make it lazy? I mean, only compute that range as we consume it? Luckily, Racket has a lazy version of it, all we need to do is set `#lang lazy` at the benning of the code file:

```scheme
; Lazy version
#lang lazy
> (range 11)
'(0 . #<promise:...7/pkgs/lazy/base.rkt:299:29>)

; Regular Racket version
#lang racket
> (range 11)
'(0 1 2 3 4 5 6 7 8 9)
```

Okay, that looks _promising_, that uses promises! ;)

[**Lazy Racket**](https://docs.racket-lang.org/lazy/) already provides all the basic list functions adapted to use those promises, so we can use our `foldl`, right?

```scheme
(foldl + 0 (range 1000000001))
```

Nope. Same problem. Let's try to figure out why.

# Tracking Memory Usage

Our assumption is that the regular Racket program is consuming too much memory to a point where the interpreter is killing it. I'm gonna use [psrecord](https://github.com/astrofrog/psrecord) to track the memory of our process and plot it to a graph. It's pretty simple to run it in a debian-based system.


## Installing psrecord

```shell
$ pip3 install psrecord
$ sudo apt-get install python3-matplotlib python3-tk
```

Then I created a shell script `plotprocess.sh` to help tracking it:

```shell
#!/bin/bash
echo "Plotting process '$2' to file '$1'..."
$2 &P1=$!
psrecord $(pgrep ${2%% *}) --interval 0.05 --plot $1
P2=$!
wait $P1 $P2
echo "Done"
```

## Tracking 
