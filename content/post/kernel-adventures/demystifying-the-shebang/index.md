---
title: "Demystifying the #! (shebang): Kernel Adventures"
description: "Clarifying the shebang (#!) mechanism: A step-by-step look using strace and kernel code shows how Linux handles script execution directly, revealing the shell isn't involved initially."
date: 2025-04-07
tags:
 - kernel-adventures
 - linux
 - investigation
 - c
 - os
---

From my first experience creating a shell script, I learned about the [**shebang**](https://en.wikipedia.org/wiki/Shebang_(Unix)) (`#!`), the special first line used to specify the interpreter for executing the script:

```shell
#! /usr/bin/sh
echo "Hello, World!"
```

So that you can just invoke it with `./hello.sh` and it will run with the specified interpreter, assuming the file has execute permissions.

Of course, the shebang isn't limited to shell scripts; you can use it for any script type:

```python
#! /usr/bin/python3
print("Hello, World!")
```

This is particularly useful because many bundled Linux utilities are actually scripts. Thanks to the shebang, you don't need to explicitly invoke their interpreters. For example, there are two (very confusing) programs to create a user on Linux: `useradd` and `adduser`. One of them is the actual program that will create the user in the system, the other one is a utility that will create the user, the home directory and configure the user for you. Since I never remember which one is which, a good way to check is using the utility `file`:

```shell
$ file $(where useradd)
/usr/sbin/useradd: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2 (...)
/sbin/useradd:     ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2 (...)

$ file $(where adduser)
/usr/sbin/adduser: Perl script text executable
/sbin/adduser:     Perl script text executable

```

Ok, we know that `addser` is the tool we want to use, because it's more user-friendly and generally does what you'd expect when adding a user. And yes, if you check how it starts:

```shell
$ head -n 1 /usr/sbin/adduser
#! /usr/bin/perl
```

I had always assumed the _shell_ used the shebang as a hint, but that's incorrect! **This functionality is actually handled directly by the Linux Kernel.**

# Tracking the kernel execution

One good way to track any executable in Linux is using `strace`, which traces all the system calls made by a process:

```shell
$ strace ./test.sh
execve("./test.sh", ["./test.sh"], 0x7ffed15d9828 /* 33 vars */) = 0
brk(NULL)                               = 0x59aea5a28000
mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x78ee2be49000
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
(...)
```

Interesting, the call to `test.sh` goes straight into `execve`, the syscall to start running a program from a file. This implies the kernel itself is _responsible for finding_ the correct interpreter and executing it.

If we start digging into the kernel code, we can see that the entry point for the `execve` syscall is in the function `do_execveat_common`, found in [`fs/exec.c`](https://github.com/torvalds/linux/blob/master/fs/exec.c#L1967). It starts by creating a `struct linux_binprm *bprm;` which means "binary program", then performing some checks, and eventually calling `bprm_execve`:

```c
retval = bprm_execve(bprm);
```

`bprm_execve` then proceeds to `exec_binprm`, which will then eventually invoke `search_binary_handler`. This function is responsible for identifying the file's executable format. It starts with `retval = prepare_binprm(bprm)` and following that function, we realize it's actually copying part of the contents of the file into the `bprm->buf`:

```c
/*
 * Fill the binprm structure from the inode.
 * Read the first BINPRM_BUF_SIZE bytes
 *
 * This may be called multiple times for binary chains (scripts for example).
 */
static int prepare_binprm(struct linux_binprm *bprm)
{
	loff_t pos = 0;

	memset(bprm->buf, 0, BINPRM_BUF_SIZE);
	return kernel_read(bprm->file, bprm->buf, BINPRM_BUF_SIZE, &pos);
}
```

`BINPRM_BUF_SIZE` is _256_ in `include/linux/binfmts.h`

Then it proceeds to look through a list of formats and checks which one the current program is:

```c
list_for_each_entry(fmt, &formats, lh) {
	if (!try_module_get(fmt->module))
		continue;
	read_unlock(&binfmt_lock);

	retval = fmt->load_binary(bprm);

	read_lock(&binfmt_lock);
	put_binfmt(fmt);
	if (bprm->point_of_no_return || (retval != -ENOEXEC)) {
		read_unlock(&binfmt_lock);
		return retval;
	}
}
```

Those format modules are:

- binfmt_elf.c
- binfmt_elf_fdpic.c
- binfmt_flat.c
- binfmt_misc.c
- binfmt_script.c

And they all are responsible for registering themselves so `search_binary_handler` test each one of them. We know that [ELF](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format) is the traditional binary format that Linux uses, [ELF FDPIC](https://cateee.net/lkddb/web-lkddb/BINFMT_ELF_FDPIC.html) is an extension to ELF, FLAT binaries are just the instructions without any specific system configuration ([this question](https://stackoverflow.com/questions/1283342/executing-a-flat-binary-file-under-linux) explains a bit), SCRIPT is the format that interprets our shebang, but what really caught my eye was MISC.

According to the [official Kernel Admin Guide](https://docs.kernel.org/admin-guide/binfmt-misc.html):

>This Kernel feature allows you to invoke almost (for restrictions see below) every program by simply typing its name in the shell. This includes for example compiled Java(TM), Python or Emacs programs. To achieve this you must tell binfmt_misc which interpreter has to be invoked with which binary. Binfmt_misc recognises the binary-type by matching some bytes at the beginning of the file with a magic byte sequence (masking out specified bits) you have supplied. Binfmt_misc can also recognise a filename extension aka `.com` or `.exe`.

It's another way to tell the Kernel what interpreter to run when invoking a program that's not native (ELF). For scripts (text files) we mostly use a shebang, but for byte-coded binaries, such as Java's JAR or Mono EXE files, it's the way to go!

Returning to our shebang investigation, let's examine `fs/binfmt_script.c`. Checking its registration mechanism near the end of the file reveals some key information:

```c
core_initcall(init_script_binfmt);
module_exit(exit_script_binfmt);
MODULE_DESCRIPTION("Kernel support for scripts starting with #!");
MODULE_LICENSE("GPL");
```

There's the module description (yep, shebang is not an official term), then a `core_initcall` call pointing to `init_script_binfmt`:

```c
static int __init init_script_binfmt(void)
{
	register_binfmt(&script_format);
	return 0;
}
```

That registers the `script_format` object, which is defined like this:

```c
static struct linux_binfmt script_format = {
	.module		= THIS_MODULE,
	.load_binary	= load_script,
};
```

And when we examine the `load_script` function, boom:

```c
static int load_script(struct linux_binprm *bprm)
{
	const char *i_name, *i_sep, *i_arg, *i_end, *buf_end;
	struct file *file;
	int retval;

	/* Not ours to exec if we don't start with "#!". */
	if ((bprm->buf[0] != '#') || (bprm->buf[1] != '!'))
		return -ENOEXEC;
(...)
```

There the check is!

This function is very well-commented, detailing almost every step, so I recommend reading the source code [here](https://github.com/torvalds/linux/blob/e48e99b6edf41c69c5528aa7ffb2daf3c59ee105/fs/binfmt_script.c#L34). Essentially, it reads the first line, parses the interpreter path (and any arguments), opens the interpreter's executable file, and assigns a reference to it to `bprm->interpreter`.

Back in `exec_binprm`, it will check for if an interpreter (from script or misc binary formats) was found, then if so:

```c
(...)
ret = search_binary_handler(bprm);
if (ret < 0)
	return ret;
if (!bprm->interpreter)
	break;

exec = bprm->file;
bprm->file = bprm->interpreter;
bprm->interpreter = NULL;

exe_file_allow_write_access(exec);
if (unlikely(bprm->have_execfd)) {
	if (bprm->executable) {
		fput(exec);
		return -ENOEXEC;
	}
	bprm->executable = exec;
} else
	fput(exec);
(...)
```

If an interpreter _is_ found, `bprm->file` is updated to point to the interpreter's file (replacing the script file), and the reference count for the original script file (`exec`) is decremented via `fput(exec)`.

So, a single `execve` syscall on the script file triggers the kernel to: open the script, detect the `#!`, find and open the specified interpreter, and finally load and execute the _interpreter_, passing the script path as an argument. The kernel effectively replaces the process image with the interpreter's.

# But I can run a shellscript without #!

That's true. You don't really need #! to run shellscripts, but that's a fallback mechanism implemented by the shell, rather than the kernel, for example, if you try to strace the execution of a shell script lacking a shebang:

```shell
$ cat test.sh
echo "Hello, World!"

$ ./test.sh
Hello, World!

$ strace ./test.sh
execve("./test.sh", ["./test.sh"], 0x7ffd9a1afcf0 /* 33 vars */) = -1 ENOEXEC (Exec format error)
strace: exec: Exec format error
+++ exited with 1 +++
```

It will fail with `ENOEXEC (Exec format error)`, since there's no indication of format for that file.

To observe the shell's fallback behavior, we can trace a _new_ shell instance invoking the script. We use `sh -c './test.sh'` to ensure the child shell attempts the `execve`, rather than the parent shell interpreting the script directly. We'll use `strace` with `-f` (to follow child processes) and filter for key syscalls:

```shell
strace -e trace=execve,openat,read,close -f sh -c "./test.sh"
```

If there's a `#!` in `test.sh`, it will return this:

```shell
$ cat test.sh
#! /usr/bin/sh
echo "Hello, World!"

$ strace -e trace=execve,openat,read,close -f sh -c "./test.sh"
execve("/usr/bin/sh", ["sh", "-c", "./test.sh"], 0x7ffd51f86418 /* 33 vars */) = 0
(...)
strace: Process 2522303 attached
[pid 2522303] execve("./test.sh", ["./test.sh"], 0x5ec40c994540 /* 33 vars */) = 0
(...)
[pid 2522303] openat(AT_FDCWD, "./test.sh", O_RDONLY) = 3
[pid 2522303] close(3)                  = 0
[pid 2522303] read(10, "#! /usr/bin/sh\necho \"Hello, Worl"..., 8192) = 36
Hello, World!
[pid 2522303] read(10, "", 8192)        = 0
[pid 2522303] +++ exited with 0 +++
(...)
```

If no `#!` is found:

```shell
$ cat test.sh
echo "Hello, World!"

$ strace -e trace=execve,openat,read,close -f sh -c "./test.sh"
execve("/usr/bin/sh", ["sh", "-c", "./test.sh"], 0x7ffd4de7e798 /* 33 vars */) = 0
(...)
strace: Process 2524967 attached
[pid 2524967] execve("./test.sh", ["./test.sh"], 0x651ce522f540 /* 33 vars */) = -1 ENOEXEC (Exec format error)
[pid 2524967] openat(AT_FDCWD, "./test.sh", O_RDONLY|O_NOCTTY) = 3
[pid 2524967] read(3, "echo \"Hello, World!\"\n", 128) = 21
[pid 2524967] close(3)                  = 0
[pid 2524967] execve("/bin/sh", ["/bin/sh", "./test.sh"], 0x651ce522f540 /* 33 vars */) = 0
(...)
[pid 2524967] openat(AT_FDCWD, "./test.sh", O_RDONLY) = 3
[pid 2524967] close(3)                  = 0
[pid 2524967] read(10, "echo \"Hello, World!\"\n", 8192) = 21
Hello, World!
[pid 2524967] read(10, "", 8192)        = 0
[pid 2524967] +++ exited with 0 +++
(...)
```

After filtering the output, it's clear that in the first case (with shebang), it's doing the initial `execve`for the shell instance we're creating, then another `execve` for `test.sh` and do all the process we described before. In the second case (no shebang), the child process's `execve` on `./test.sh` fails with `ENOEXEC`. The parent shell (`sh -c`) catches this error. It then likely uses `openat` and `read` to examine the file. Detecting it's likely a shell script, it then explicitly executes `/bin/sh ./test.sh` via a _second_ `execve` call.

# Bonus: Dealing with Permissions

We found out that the kernel runs the scripts through its own `execve` syscall assuming it contains a `#!` _and_ has execute permission set. But where is that permission checked?

If we try to invoke a script that doesn't have execute permissions, we'll get this:

```shell
$ ./test.sh
zsh: permission denied: ./test.sh
```

But it doesn't give off too much. However, if we `strace` it again:

```shell
$ strace ./test.sh
execve("./test.sh", ["./test.sh"], 0x7ffd2b4a52d0 /* 33 vars */) = -1 EACCES (Permission denied)
strace: exec: Permission denied
+++ exited with 1 +++
```

It returns the error code and description from the syscall: `EACCES (Permission denied)`. **Error codes are always a good start point**. Searching for `EACCES` in `fs/exec.c` leads us to the check within the `do_open_execat` function

```c
if (WARN_ON_ONCE(!S_ISREG(file_inode(file)->i_mode)) ||
	path_noexec(&file->f_path))
	return ERR_PTR(-EACCES);
```

Tracing the call stack back from `do_open_execat`, we find it's called during the setup of the `bprm` structure within `do_execveat_common`, the entrypoint to the `execve` syscall:

```c
bprm = alloc_bprm(fd, filename, flags);
if (IS_ERR(bprm)) {
	retval = PTR_ERR(bprm);
	goto out_ret;
}
```

Now, understanding how `path_noexec` checks the _execute_ permission in the file involves a lot of other stuff like understanding how the kernel deals with the filesystem. But that'll be a future post.
