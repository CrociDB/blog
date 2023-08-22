---
title: "Installing Portaudio for Racket on Windows"
description: "A quick guide to instaling Portaudio for Racket on Windows"
date: 2023-08-22T11:57:44+02:00
draft: false
tags:
 - lisp
 - audio-programming
 - racket
---

I recently wanted to play with some procedural audio generation in Racket. So, after some research, I found that there's a [port](https://docs.racket-lang.org/portaudio/index.html) of the cross-platform audio I/O library [Portaudio](https://www.portaudio.com/). However, installing it through `raco` was not doing the job for me, on Windows. I investigated the issue and figure a problem with the compiled libraries that were out of date with the Racket interface and then I rebuilt it and submitted to another repository. I created a [Pull Request](https://github.com/jbclements/portaudio-x86_64-win32/pull/1) but still hasn't been accepted.

There were very few people with the same issue, I imagine manipulating raw audio buffers in interpreted languages with garbage collectors is not really popular, but I hope this can help _someone_.

The main `portaudio` repo is fine, it's just the dependent repo with the specific binary library for the Windows system that is broken, so we need to explicitly install that from [my repository](https://github.com/CrociDB/portaudio-x86_64-win32) first:

```
raco pkg install https://github.com/CrociDB/portaudio-x86_64-win32.git
```

Then finally install `portaudio`. The dependency will be ignored, since it's already installed:

```
raco pkg install portaudio
```

Packets can be installed through **DrRacket** as well, just by hitting `File -> Install Package`. But remember to do in the same order.

## Running the Example

There are examples for both realtime streams and pre-allocated buffer play on the [port docs page](https://docs.racket-lang.org/portaudio/index.html#%28part._.Playing_.Sounds%29). And playing the copy version should work out of the box:

```racket
#lang racket
 
(require portaudio
          ffi/vector)
 
(define pitch 426)
   
  (define sample-rate 44100.0)
  (define tpisr (* 2 pi (/ 1.0 sample-rate)))
  (define (real->s16 x)
     (inexact->exact (round (* 32767 x))))
   
  (define vec (make-s16vector (* 88200 2)))
  (for ([t (in-range 88200)])
     (define sample (real->s16 (* 0.2 (sin (* tpisr t pitch)))))
       (s16vector-set! vec (* 2 t) sample)
         (s16vector-set! vec (add1 (* 2 t)) sample))
   
  (s16vec-play vec 0 88200 sample-rate)
  ```


