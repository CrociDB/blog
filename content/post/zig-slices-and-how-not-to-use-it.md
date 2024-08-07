---
title: "Zig Slices and how not to use it"
description: "I've hit an undefined behavior with Zig and how I learned about Slices"
date: 2024-08-07
draft: false
tags:
 - programming
 - zig
 - investigation
---

I started another [CodeCrafters](https://app.codecrafters.io/r/healthy-otter-219488) project: the BitTorrent, but this time in Zig. I've been wanting to learn Zig for at least a year and I thought that would be a good opportinity. It starts with a very interesting challenge, implement the [Bencode](https://en.wikipedia.org/wiki/Bencode) encoding algorithmn, used to serialize based on data-types. At some point I stumbled onto an issue that seemed a bit non-sense.

To simplify, it was a case with this function that is included in the base project:

```zig
pub fn main() !void {
    // ...
    const decodedStr = try decode("3:abc");

    var string = std.ArrayList(u8).init(allocator);
    try std.json.stringify(decodedStr.*, .{}, string.writer());
    const jsonStr = try string.toOwnedSlice();
    try stdout.print("{s}\n", .{jsonStr});
}

pub fn decode(encodedValue: []const u8) !*const []const u8 {
    if (encodedValue[0] >= '0' and encodedValue[0] <= '9') {
        // ...
        return &encodedValue[1..]; // gets a slice of the slice, cutting the first character
    } else {
        try stdout.print("Only strings are supported at the moment\n", .{});
    }

    return error.NotSupported;
}
```

I simplified a bit, but it's basically a function that accepts a _slice of string_ and return a _pointer a another slice of that string_. After that, it converts thatstring into json using the standard library and prints it. I noticed that, when I removed the line `try stdout.print("...");` line from the `decode` function, it would print an empty string instead. I was confident that `decode` returned a valid string regardless of a print call that wasn't ever being executed, but for some reason `stringify` would return different values based on that.

The experienced Zig programmer probably already seen the issue, but it took me some time. I was set to debug it, but since I still couldn't make `lldb` work with my neovim setup (with dap), I had to do it by hand. And that's when I learned my first lesson (well, second, first is _read the fucking manual_): **Slices are pointers**. But not just any pointer:

```zig
(lldb) n
Process 5163 stopped
* thread #1, name = 'main', stop reason = step over
    frame #0: 0x0000000001035e94 main`main.main at main.zig:8:40
   5    pub fn main() !void {
   6        const decodedStr = try decode("3:abc");
   7
-> 8        var string = std.ArrayList(u8).init(allocator);
   9        try std.json.stringify(decodedStr.*, .{}, string.writer());
   10       const jsonStr = try string.toOwnedSlice();
   11       try stdout.print("{s}\n", .{jsonStr});
(lldb) p decodedStr
([]u8 *) 0x00007fffffffe400
(lldb) p *decodedStr
([]u8)  (ptr = ":abc", len = 4)
(lldb) p decodedStr->ptr
(unsigned char *) 0x000000000101f116 ":abc"
```

Going part by part: `decodedStr` is a raw pointer to `[]u8`, a slice of unsigned 8bit ints. The slice is a _structure_ that holds a pointer and the length of the slice, a bit similar to the [Simple C Vector I described here](/post/simple-vector-implementation-in-c/). Which means that, `&encodedValue[1..]` actually creates a slice in the stack and then gets the address of it. So `decodedStr` was pointing to an object in the stack, so when the next function in main got executed, `var string = std.ArrayList(u8).init(allocator);`, the stack was being overwritten, so dereferencing it on the call of `stringify` would just bring trash. Funny enough, it wouldn't _always_, because when the `try stdout.print("...` was present, the message string was also stored in the stack, making the stackframe just large enough so that slice wouldn't be wiped out by the ArrayList initialization.

The experienced C programmer is now laughing, because returning a pointer to an object in that current stack frame is something that C compilers usually emit Warnings for and one would think that Zig does the same, at least. Or even not allow it at all.

Nothing like shooting your own foot so you understand how your gun works. I think I actually learned some valuable stuff about Zig. It's more of a "dev diary" kind of post, where I just write about my findings in a more raw way, so I can use it as a reference later. And in that case, these are some links that I needed to go through to learn it:

 - [GDB to LLDB command map](https://lldb.llvm.org/use/map.html): I've debugged with GDB before, but never with LLDB. Good source to learn it by comparison.
 - [Zig Guide: Slices](https://zig.guide/language-basics/slices/): the official guide to Slices. I should've read it in the moment I saw the code using the brackets.
 - [Zig Crash Course](https://ikrima.dev/dev-notes/zig/zig-crash-course/): interesting resource. Worth taking a read if you're just starting with the language.
