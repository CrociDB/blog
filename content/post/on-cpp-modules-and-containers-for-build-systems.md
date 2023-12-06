---
title: "On C++ Modules and Containers for Build Systems"
description: ""
date: 2023-12-06T01:58:51+02:00
draft: false
tags:
 - build
 - programming
 - annileen
---

We recently rebooted the development of our toy game engine [annileen](https://github.com/CrociDB/annileen). [Teo](https://teodutra.com/) really wanted to make it with modern C++ and he spent some good time [converting it to use C++20 modules](https://teodutra.com/annileen/annileen-devlog/game-engine/graphics-programming/cpp/cpp20/2023/02/27/Annileen-Devlog-2/). However, the current state of modules in the compilers is a bit complicated, and that requires a big effort to make it compile in any platform but Windows.

I also wasn't very happy with all the libraries we were using in the engine and decided to start from scratch and make a renderer with Vulkan and not BGFX. After all, the main objective of this project is to teach myself graphics programming and engine development, if I used third party libraries for every aspect of the engine, I'm basically gluing stuff together.

Then I started by researching how to compile modules in Linux. That's a really hard topic, because at first all you find is "module support is experimental" in both GCC and Clang. And that's for a reason, modules require a new step in the build. You need to compile every module in order of dependency. Before modules, you would compile every translation unit on their own and then link them together, so it didn't matter the order. It was easier to create the makefile, now, however, your makefile needs to be in order. So either you're creating a makefile manually, or the tool that generates it needs to figure out dependencies before. That changes the whole build system you're using.

In the [old version of annileen](https://github.com/CrociDB/annileen), we were using **premake5**, which is a great tool to generate projects for different platforms/compilers. But guess, it doesn't support modules and doesn't have any short-time plans. I thought about taking the time to implement it and submit a PR, but if I took that approach, I would never start the new version of the engine.

In CMake's text explaining how they implemented their [support for modules](https://www.kitware.com/import-cmake-c20-modules/), that sounds way more complex than I thought. Microsoft usually implements new C++ specifications way before the other compilers, I guess being a centralized closed-software helps a lot on that. Visual Studio natively supports modules, every time you compile it will figure out automatically the order of module compilation, that's why Teo's version of annileen using modules were working well on Windows. However, other compilers will only deal with modules in very specific versions and you need to use many experimental command line flags, that are different for each compiler, in order to build each module. The functionality to detect dependecies are mostly implemented in these compilers, but since it needs to be made in a different step than the compilation, there must be some sort of communication between the build system and the compiler, and each experimental version of each compiler would do it differently.

CMake recently proposed a [standard for the source files dependency description](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p1689r5.html), based on the [Fortran module support](https://www.kitware.com/import-cmake-c20-modules/). That's important because the build tools need to query the compiler with all the source code and get the result before generating the Makefile (or Ninja) with the correct dependency order. In theory, GCC and Clang already have patches for this standard, but in practice it's a different story. In GCC it's only available in GCC13, released just a few months before the writing of this post. Clang, however, it's only officially available after Clang17. Other option would be get patched versions of these compilers, but if they don't provide a binary version for that, you'll have to build it.

This [article by the CMake team](https://www.kitware.com/import-cmake-c20-modules/) explain well how to get modules working on Clang 17. Another good resources to start is this [text by Victor Zverovich](https://www.zverovich.net/2023/04/17/simple-cxx20-modules.html) as well as the [CMake library](https://github.com/vitaut/modules) he put together, that supports GCC. That's what we're using on the engine.

But that was not without issues. One problem I found right off the bat was that, if you're on Linux, compiling with Clang will use the default `libstdc++` on the system, so you're basically bound to what your current version of GCC has available in terms of modern C++. `std::format` was the first issue, turns out its support is still limited in recent versions of the compilers. Another issue is that modules in CMake only supports Ninja (specifically version 1.11) for now. Ninja a faster `make` tool made by Google that I just learned existed because of this.

Then I thought about containerizing the build system. Since the project requires a specific version of CMake, a specific version of Clang and a specific version of Ninja, that sounded like an easy way to keep the build simple. I've seen other projects doing it, it must be a good idea. So I created an image based on Arch Linux, especially because the rolling release nature of the distro could be easier for me to keep it updated. The Dockerfile was:

```Dockerfile
# syntax=docker/dockerfile:1
FROM archlinux:base-devel

WORKDIR /annileen

RUN pacman -Sy unzip wget --noconfirm
RUN wget https://github.com/ninja-build/ninja/releases/download/v1.11.1/ninja-linux.zip
RUN unzip ninja-linux.zip
RUN mv ninja /usr/bin
RUN pacman -Sy cmake=3.27.7 --noconfirm
RUN wget https://github.com/llvm/llvm-project/releases/download/llvmorg-17.0.2/clang+llvm-17.0.2-x86_64-linux-gnu-ubuntu-22.04.tar.xz
RUN tar xvf clang+llvm-17.0.2-x86_64-linux-gnu-ubuntu-22.04.tar.xz
RUN mv clang+llvm-17.0.2-x86_64-linux-gnu-ubuntu-22.04 /usr/lib/clang

ENV PATH "$PATH:/usr/lib/clang/bin"

CMD ./build.sh
```

The `build.sh` file was simple like this:

```shell
mkdir build
cd build
cmake .. -GNinja
ninja
```

And, in order to share the current project folder with the container, I had another shellscript `dockerbuild.sh`:

```shell
sudo docker build -t annibuild .
sudo docker run -v $(pwd):/annileen annibuild
```

And it worked pretty well. In theory only calling `dockerbuild.sh` would start the container, install the dependencies, run CMake and finally build it. But first time I tried to run it I figured how flawed this whole container build idea was:

```shell
./anni: /lib/x86_64-linux-gnu/libstdc++.so.6: version `GLIBCXX_3.4.32' not found (required by ./anni)
```

Building the project in your machine ultimately means that it should run on it. So when I used the container, it's using the `libstdc++` available in that container, which is different than my current machine. Of course I could tweak the container as much as possible, use older libraries, but that will never be a 100% compatible with the host system. So I dropped that solution and focused on writing good build docs.

The container build is still, however, valuable for CI. I'll eventually setup a VPS server with some CI system for build and automated tests. I'm currently leaning towards [FlowCI](https://flowci.github.io/), but any other suggestion would be appreciated.

There's still a lot to learn from this project. I can already foresee that every new feature we implement using modern C++ stuff will yield new problems, but we'll try to learn with it and I'll bring more issues to this blog as I'm having them.

