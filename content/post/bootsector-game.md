---
title: "Writing a game for the boot sector"
description: "How to create a game to run on the bootsector. The whole game must have 512 bytes, but you don't need an operating system to run it."
date: 2021-09-08T17:57:44+02:00
draft: false
tags:
 - lowlevel
 - gamedev
---

I was recently exposed to the underworld of boot sector games, thanks to great book **Programming Boot Sector Games** by Oscar Toledo, aka **nanochess**. They are tiny little games, up to 512 bytes of machine code, that run on the bootsector of a disk, the space reserved for the bootloaders to initialize the operating system.

You may think that 512 bytes are not enough to write a game, and it's not a bad assumption, however there are people taking the challange seriously. The 8-bit Guy has a nice [introductory video](https://www.youtube.com/watch?v=1UzTf0Qo37A) about the subject, and although he doesn't look very excited with the games, he says at the end something I completely agree: "people who write these games, they have more fun making them than actually playing". [Oscar Toledo](https://github.com/nanochess) wrote [Space Invaders](https://github.com/nanochess/Invaders), Chess, [Flappy Bird](https://github.com/nanochess/fbird) and [Pac Man](https://github.com/nanochess/Pillman) and they're very impressive in my opinion.

I confess my initial motivation to get the book was to learn some assembly and enjoy a different challange. The introduction is pretty good and the examples very well explained.  So after a third of the book, I decided to write my own game: a 2048 clone.

I'll describe here a little about the development of this project, but first you can [play it live](https://crocidb.github.io/retro2048/), using a x86 javascript emulator or watch the video I made:

{{< youtube lexTohTMSNw >}}

## Hello, World!

There's two things to consider when writing a bootsector program: it needs to be written in 16bit real mode 8086/8088 assembly language and fit 512 bytes. There's no libraries, just plain cpu instructions on registers and access to the BIOS services through interrupts.

I took the approach of writing it as a real mode dos program, but without the size concern at first, then later try to optimize it--and even cut features if necessary--and making it a bootsector game.

I started with a basic text mode program, since the 2048 is basically a text (number really) game:

```asm
    org 0x0100                  ; The entry address to copy the code to;
                                ; this changes when setting it to run from boot sector

start:
    mov ax, 0x0002              ; Set 80-25 text mode
    int 0x10

    mov ax, 0xb800              ; Segment for the video data
    mov es, ax
    cld

exit:
    int 0x20

```

It's simple to assemble that with NASM:

```
nasm -f bin -o game.com game.asm
```

This will generate a `game.com` file, a DOS executable. If you run that on DOSBox you'll notice that it doesn't do much, basically just sets the text mode, acquires the pointer to the video memory and then exits.

## The text mode

BIOS support a few graphics modes. We're using the 80-25 text mode, which means it supports 80 columns and 25 lines of characters. So the video memory from `0xB800` to `0xC7A0` will store two bytes for each character, one for formatting and one for the ASCII code. In order to put a character in the screen, all we need to do is copy the character data into the right position in memory. Example, this two bytes `0x2761` means character `'a'` (0x61), green background (0x20) and gray foreground (0x07). More info on the colors can be found [here](https://en.wikipedia.org/wiki/BIOS_color_attributes). 

First we need to set the text mode calling the interrupt service `0x10`, passing `AH=00` (set video mode function) and `AL=02` (the mode 02). Here you can find more [functions of service 0x10](https://en.wikipedia.org/wiki/INT_10H), and here more [information on video modes](http://minuszerodegrees.net/video/bios_video_modes.htm). Right after that we acquire the pointer to the video data and store it onto the _Extended Segment_ register: `ES`. 

> Segment registers are the two least significant bytes when addressing the memory, so whenever you access memory you need a segment and an offset, example: `ds mov [bp], 0x0000`, will move the value `0x0000` to the memory address `DS:BP`, being `DS` the segment and `BP` the offset.

After that, we're good to display some text on screen. Or display some simple graphics, like this function that displays a color filled box:

```asm
    ;
    ; Draw box function
    ; Params:   [bp+2] - row offset
    ;           [bp+4] - column offset
    ;           [bp+6] - box dimensions
    ;           [bp+8] - char/Color
    ;
draw_box:
    mov bp, sp                      ; Store the base of the stack, to get arguments
    xor di, di                      ; Sets DI to screen origin
    add di, [bp+2]                  ; Adds the row offset to DI

    mov dx, [bp+6]                  ; Copy dimensions of the box
    mov ax, [bp+8]                  ; Copy the char/color to print
    mov bl, dh                      ; Get the height of the box

    xor ch, ch                      ; Resets CX
    mov cl, dl                      ; Copy the width of the box
    add di, [bp+4]                  ; Adds the line offset to DI
    rep stosw

    add word [bp+2], 160            ; Add a line (180 bytes) to offset
    sub byte [bp+7], 0x01           ; Remove one line of height - it's 0x0100 because height is stored in the msb
    mov cx, [bp+6]                  ; Copy the size of the box to test
    cmp ch, 0                       ; Test the height of the box
    jnz draw_box                    ; If not zero, draw the rest of the box
    ret
```

In order to invoke that function, we need to push the arguments to the stack and then call it, like this:

```asm
    ; Drawing the box
    push 0x3800                 ; Background: cyan; Foreground: blue; Character: empty
    push 0x1125                 ; Rect size 37x16 (25x11 in hex)
    push 44                     ; Offset 22 chars on left
    push 160 * 5                ; Offset 5 lines on top
    call draw_box
```

The first line of the `draw_box` function will store the pointer to the stack on `BP`, so we could retrieve the parameters from the stack. The addressing starts with `BP+2` since the last value on the stack points to the address that invoked the `call` instruction, so `ret` knows where to return to.

The most interesting line in this function probaly is: `rep stosw`. `rep` instruction will decrement CX and repeat the next instruction while CX > 0, it does pretty much the same thing as the instruction `loop`, except it executes a instruction instead of jumping to a label. `stosw`, however, will copy the value of `AX` into `[ES:DI]` and will increment the pointer in `DI`. So before that, we basically copy the char data to `AX`, add the offset to that position in `DI`, and set the amount of chars to `CX`, then `rep stosw` will do all the work for us.

Of course we still need to display some text:

```asm
    ;
    ; Print string function
    ; Params:   AH - background/foreground color
    ;           BP - string addr
    ;           CX - position/offset
    ;
print_string:
    mov di, cx                      ; Adds offset to DI
    mov al, byte [bp]               ; Copies the char to AL (AH already contains color data)
    cmp al, 0                       ; If the char is zero, string finished
    jz _0                           ; ... return
    stosw
    add cx, 2                       ; Adds more 2 bytes the offset
    inc bp                          ; Increments the string pointer
    jmp print_string                ; Repeats the rest of the string
_0:
    ret
```

In order to print a text to screen, we need to store that somewhere, and it's usually done in the end of the program using the `db` or `dw` directives. They will tell the assembler that whatever data comes after will be stored as bytes or words, respectivelly. Example:

```asm
title_string:       db " r e t r o 2 0 4 8 ", 0
```

Strings are array of bytes, since every char is as ascii code that goes from 0 to 255. We're telling the assembler to store the string ` r e t r o 2 0 4 8 `, followed by a zero, in the position `title_string` in the end of the program. Ending the string with zero is important so we know when to stop printing, it's exactly what line `cmp al, 0` of our `print_string` function is doing.

The function accepts a byte for format (background and foreground), the address of the text we want to print and finally the offset in the screen, so if we want to print the text on the position 40,04 (column 40 of line 05), that means our offset has to be `(80 * 4 + 40) * 2`, the number of lines multiplied by the number of columns plus the columns in this line, all multiplied by two, because it's two bytes per character.

```asm
    ; Game title
    mov ah, 0x67                ; Background: brown; Foreground: Light gray
    mov bp, title_string        ; Copying the address to the text
    mov cx, 62                  ; 62 => (0, 31)
    call print_string

    ; Drawing the box
    push 0x3800
    push 0x1125                 ; Rect size 37x16 (25hx11h)
    push 44                     ; Offset 22 chars on left
    push 160 * 5                ; Offset 5 lines on top
    call draw_box
```

And then we get this:

![Board](https://i.imgur.com/akxQiGl.png)

Not exactly a game, but we have something on screen!

## Boot Sector

In order to make that program run in the bootsector, first we need to change the entry address of the program. The `org` directive we put in the start of the program now changes from value `0x0100` to `0x7c00`.

```asm
    org 0x7c00
```

Then we need to force the final binary to be exactly 512 bytes, even if the code was less than that. Followed by the bootable signature in the end of the file:

```asm
    times 510-($-$$) db 0x4f
    db 0x55, 0xaa                   ; bootable signature 
```

There's no difference to assemble that for bootsector, only that we might want to name with a different extension than `com`, usually `bin`, because that won't be a DOS executable anymore, and if you try to run that, it will crash.

```
nasm -f bin -o game.bin game.asm
```

While we were testing the DOS version on DOSBox, now we need a proper i386 emulator to boot the game. I'm going for [qemu](https://www.qemu.org/download/) because it's very simple to test. Once you have that installed, you can invoke a virtual machine by typing in:

```
qemu-system-i386 -fda game.bin
```

### I want to boot on my computer

Alright, now you'll ask how to test that on an _actual_ computer. And that's the whole point of it, for sure. There's good news and bad news regarding that. The bad news is that [Intel discontinued the support for legacy boot](https://www.intel.com/content/dam/support/us/en/documents/intel-nuc/Legacy-BIOS-Boot-Support-Removal-for-Intel-Platforms.pdf) as of 2020. Good news is that you probably have a computer that still supports that. In that case, you'll need a USB stick--or a floppy disk, if you prefer.

For USB sticks, you can burn the image on the bootsector using [Rufus](https://rufus.ie/), if you're on Windows, or `dd` on a linux/unix platform. Just make sure to replace `/dev/disk1` with the actual USB device you want to burn to:

```
dd if=game.bin of=/dev/disk1 bs=512 count=1
```

Then change your prefered boot device on your BIOS, stick it in and reboot.

![Booting from a computer](https://i.imgur.com/zuJffBg.png)

## Challanges

### Algorithms

I really liked this project because I remember how much _fun_ it was implementing algorithms in assembly. All the logic for the 2048 game seem so simple to write in an imperative language, but when it comes to bring it down to instructions, it's a completely different way of thinking that I am not used to. 

Considering the movement and evaluation. Whenever you hit one of the arrow keys, up for example, you have to move all blocks from the 4 columns to an upper position and if there are two blocks of the same value, you merge them. I used a "simple" array to represent the board, where `0` is empty block:

```asm
board:
    db 0,0,0,0
    db 0,0,0,0
    db 0,0,0,0
    db 0,0,0,0
```

The movement and evaluation happens by "line", but what's the concept of line? It's an address of the initial block and an offset to the next block in that line. Check function `compute_board_line` at the [full source code](https://github.com/CrociDB/retro2048/blob/3a81340887bb062c7721fd974a8ce441fc950f5d/main.asm#L213) for the implementation. We also need an offset between one line to the next line. The description of each movement is basically:

```asm
movement_up:    db 4, 0, 1
movement_left:  db 1, 0, 4
movement_right: db -1, 3, 4
movement_down:  db -4, 12, 1
```

> In order: (1) the offset, (2) the address of the board, (3) the offset between lines.

![Addressing the board lines](https://i.imgur.com/7eV0EcI.png)

That way, checking the input and calling for each behaviour is quite simple:

```asm
_up:
    mov bp, movement_up
    jmp _movement
_left:
    mov bp, movement_left
    jmp _movement
_right:
    mov bp, movement_right
    jmp _movement
_down:
    mov bp, movement_down

_movement:
    mov al, byte [bp]
    cbw
    mov word [current_offset], ax
    mov ax, board
    add al, byte [bp+1]
    xor dx, dx
    mov dl, byte [bp+2]
    call compute_movement
```

And the function [`compute_movement`](https://github.com/CrociDB/retro2048/blob/3a81340887bb062c7721fd974a8ce441fc950f5d/main.asm#L196) will basically create a loop for the four lines and call [`compute_board_line`](https://github.com/CrociDB/retro2048/blob/3a81340887bb062c7721fd974a8ce441fc950f5d/main.asm#L213) passing the correct parameters.

That made me think in a way that I probably wouldn't if I were writing Javascript, for example.

### Size limitation

Of course having to write the complete game with such a limitation is a hell of a challenge, since 512 bytes can come quicker than you think. I had to constantly make decisions such as keep the score, the game title and randmozation for the new block out. Many of the algorithms were implemented a few times in order to make it fit to the final version of the game. And that's why I keep a DOS version of it, because it's clearly more interesting and fun.


## Conclusion

It was a lot of fun working on this project, and I was only about one third of the book when I started and since then didn't read any more of it. I have more ideas for little experiments and I'll definitely spend more time exploring it. It's the sort of retro studies that I like doing, even though it might not sound very useful at first.
