---
title: "Modding an Atari 2600 game"
description: ""
date: 2025-11-04
draft: true
tags:
 - gamedev
 - modding
 - retro
 - assembly
---

As a kid, my first video game console was an Atari 2600. Not in the seventies, it was late 90's already, but that's what I could have. If I remember the story well, it was my dad's old console, that he found and brought back to life. Back then, however, 20 years of console evolution wasn't that big of a difference. The console I asked, the SNES, was released 13 years after the Atari, but its CPU was based on the same architecture.

I had a lot of fun with it anyway. Some of the games we had were Pac-Man, Space Master X-7 and, of course, River Raid. The latter was the most special because it felt huge, infinite even, although being too difficult for me. After all those years, I was wondering what happened after the first few levels, so I thought I could try and remove the collisions.

# Disclaimer

I haven't played any Atari games in a long time, and I didn't notice, when I started this, that most of the emulatores already come with an option to disable collisions. So, this text is mostly a rewritten version of my notes while going through this. You'll follow my thought process and mistakes, but maybe you'll have fun. Remember, it's all about the process!

# First steps

After a quick research, I found a [disassembly of the game online](https://www.romhacking.net/documents/518/), made by [Thomas Jentzsch](https://www.romhacking.net/community/1322/). He's still very active in the [Atari 2600 community](https://forums.atariage.com/profile/45-thomas-jentzsch/), and has interesting [games of his own](https://github.com/thrust26) for the platform. The code is really well documented, which makes my job much easier.

Ok, first thing is: compile and run it. I followed this tutorial [Atari 2600 Programming for Newbies](https://www.randomterrain.com/atari-2600-memories-tutorial-andrew-davie-01.html) to setup my environment with DASM and Stella, the assembler and the emulator.

Then I created a Makefile to build it:

```makefile
BUILDDIR = build
NAME=RiverRaid
SRC = $(NAME).asm
BIN = $(BUILDDIR)/$(NAME).bin
LST = $(BUILDDIR)/$(NAME).lst
SYMBOLS = $(BUILDDIR)/$(NAME).sym
ATARI_LIB=/opt/dasm/machines/atari2600/

$(BIN): $(SRC)
	mkdir -p $(BUILDDIR)
	dasm $(SRC) -I$(ATARI_LIB) -f3 -o$(BIN) -l$(LST) -s$(SYMBOLS)

clean:
	rm -f $(BIN) $(LST) $(SYMBOLS)
```

Just explain a bit why the assembler generate three files, `RiverRaid.bin` is the rom, and it's all that's needed to play it; `RiverRaid.lst` is a _listing file_, the assembly after macros and addresses resolving, with the address of every instruction; and finally `RiverRaid.sym`, with the symbols, to help us debug. It also has to explicitely set the environment variable `ATARI_LIB`.

There's only one problem:


