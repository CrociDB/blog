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

Imagine you want to count the sum of the numbers from **1** to **1,000,000,000** (one billion). If you come from an imperative programming background, you might be already thinking on a loop and what type integer to hold that value. You might come up with something [similar to this](https://godbolt.org/z/Gos5Tvenn) if you're writing C:

```c
uint64_t sum = 0; 
for (int i = 1; i <= 1000000000; i++)  
    sum += i;  
printf("%lu\n", sum);
```

Good, it runs quite fast and we get the solution. But let's try to write a solution for that problem in a functional-style Python:

```python
sum(range(1,1000000001))
```

It's more elegant and simpler. I like it. But I'm not satisfied and I want to do it in Racket:

```scheme
(apply + (range 1000000001))
```

Since Racket doesn't have a `sum` function, I use `apply` and pass the function `+` to it. But when I try to run in DrRacket, I get this error:

![Memory Error in DrRacket executing the naive approach](images/racket-apply-memory-error.jpg)

It makes sense, because the `apply` function basically gets every element in the list and apply as parameters to pass to the function specified. Example:

```scheme
(apply + (range 10))
```

Will expand to:

```scheme
(+ 0 1 2 3 4 5 6 7 8 9)
```

And considering that Racket uses big numbers as the default numeric type, I can imagine that it'll take a lot of memory to expand a million values. But that's unecessary, let's try a different way, let's fold that list:

```scheme
(foldl + 0 (range 1000000001))
```

> `foldl` stands for `Fold Left`, it will _reduce_ the list using the _procedure_ passed in the first parameter with the _accumulator_ in the second parameter. In that example, it will call the `+` procedure with 0 and the first element in the list, then update the _accumulator_ for the next element in the list and so on.

Oh no. Same memory problem. But expected again, after all, the `range` function creates a list with the numbers within that range, then we have the exact same problem as before. Which means that it's the `foldl` that is breaking our program, but the `range` itself. But what if we could make it lazy? I mean, only compute that range as we consume it? Luckily, Racket has a lazy version of it, all we need to do is set `#lang lazy` at the beginning of the code file:

```scheme
; Regular Racket version
#lang racket
> (range 11)
'(0 1 2 3 4 5 6 7 8 9)

; Lazy version
#lang lazy
> (range 11)
'(0 . #<promise:...7/pkgs/lazy/base.rkt:299:29>)
```

Okay, that looks _promising_, that uses promises! ;)

[**Lazy Racket**](https://docs.racket-lang.org/lazy/) already provides all the basic list functions adapted to use those promises, so we can use our `foldl`, right?

```scheme
(foldl + 0 (range 1000000001))
```

Argh! Memory error again. Let's try to figure out why.

# Tracking Memory Usage

We know that the regular Racket program is consuming too much memory to a point where the interpreter is killing it. I'm gonna use [psrecord](https://github.com/astrofrog/psrecord) to track the memory of our process and plot it to a graph. It's pretty simple to run it in a debian-based system.

## Setting up psrecord

```shell
$ pip3 install psrecord
$ sudo apt-get install python3-matplotlib python3-tk
```

Then I created a shell script `plotprocess.sh` to help tracking it:

```shell
#!/bin/bash
$* 
&P1=$!
graphfile="$(date +%s).png"
psrecord $P1 --interval 0.05 --plot $graphfile
P2=$!
wait $P1 $P2
echo "Graph plot in '$graphfile'" 
```

Now we only need to call `$ ./plotprocess.sh process` to start the process and plot the memory/cpu of it.

## Tracking

I created two files with both regular and lazy racket code for the foldl:

```scheme
; foldl-naive.rkt
#lang racket
(foldl + 0 (range 1000000001))

; foldl-lazy.rkt
#lang lazy
(foldl + 0 (range 1000000001))
```

So now I can track the memory usage of both of them: 

```shell
$ ./plotprocess.sh racket foldl-naive.rkt
Attaching to process 10866
./plotprocess.sh: line 6: 10866 Killed               $*
Graph plot in '1680099684.png'

$ ./plotprocess.sh racket foldl-lazy.rkt
Attaching to process 10914
./plotprocess.sh: line 6: 10914 Killed               $*
Graph plot in '1680100084.png'
```

![Memory and CPU from the execution of the naive and lazy foldl programs](images/naive-lazy-foldl.jpg)

Well, the memory usage of the process is clearly rising up to 700mb (which I don't know if it's a setting in the Racket interpreter or something in my linux server). Then it's killed. But there's an important thing to note here: the times. The naive version gets killed after around 3 seconds, whereas the lazy one goes over 10 seconds. That tells me that the lazy version actually getting something done, but is probably getting caught by the garbage collector.

In fact, I decided to test it with the `trace` library that Racket provides. The only problem is it only traces custom procedures. So I had to reimplement `foldl`:

```scheme
(require racket/trace)

(define (lfoldl f v l)
  (if (empty? l)
      v
      (lfoldl f (f (car l) v) (cdr l))))

(trace lfoldl)
```

Now running it for both regular racket and lazy racket gets me me this:

```scheme
> #lang racket
> (lfoldl + 0 (range 1000000001))
Interactions disabled; out of memory

> #lang lazy
> (lfoldl + 0 (range 1000000001))
>(lfoldl #<procedure:+> 0 '(0 . #<promise:...7/pkgs/lazy/base.rkt:299:29>))
  <#<promise:...e/pkgs/lazy/base.rkt:364:27>
>(lfoldl #<procedure:+> 0 '(1 . #<promise:...7/pkgs/lazy/base.rkt:299:29>))
  <#<promise:...e/pkgs/lazy/base.rkt:364:27>
>(lfoldl #<procedure:+> 1 '(2 . #<promise:...7/pkgs/lazy/base.rkt:299:29>))
  <#<promise:...e/pkgs/lazy/base.rkt:364:27>
>(lfoldl #<procedure:+> 3 '(3 . #<promise:...7/pkgs/lazy/base.rkt:299:29>))
  <#<promise:...e/pkgs/lazy/base.rkt:364:27>
>(lfoldl #<procedure:+> 6 '(4 . #<promise:...7/pkgs/lazy/base.rkt:299:29>))
  <#<promise:...e/pkgs/lazy/base.rkt:364:27>
>(lfoldl #<procedure:+> 10 '(5 . #<promise:...7/pkgs/lazy/base.rkt:299:29>))
  <#<promise:...e/pkgs/lazy/base.rkt:364:27>
>(lfoldl #<procedure:+> 15 '(6 . #<promise:...7/pkgs/lazy/base.rkt:299:29>))
  <#<promise:...e/pkgs/lazy/base.rkt:364:27>
>(lfoldl #<procedure:+> 21 '(7 . #<promise:...7/pkgs/lazy/base.rkt:299:29>))
  <#<promise:...e/pkgs/lazy/base.rkt:364:27>
>(lfoldl #<procedure:+> 28 '(8 . #<promise:...7/pkgs/lazy/base.rkt:299:29>))
  <#<promise:...e/pkgs/lazy/base.rkt:364:27>
>(lfoldl #<procedure:+> 36 '(9 . #<promise:...7/pkgs/lazy/base.rkt:299:29>))
  <#<promise:...e/pkgs/lazy/base.rkt:364:27>
>(lfoldl #<procedure:+> 45 '(10 . #<promise:...7/pkgs/lazy/base.rkt:299:29>))
  <#<promise:...e/pkgs/lazy/base.rkt:364:27>
>(lfoldl #<procedure:+> 55 '(11 . #<promise:...7/pkgs/lazy/base.rkt:299:29>))
```

Until a point where it gets killed due to memory usage. 

In that case, maybe we can invoke the garbage collector at every step of the fold!

```scheme
```

