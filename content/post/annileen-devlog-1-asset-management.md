---
title: "Annileen Devlog #1 - Asset Management"
description: "How the asset management works in Annileen and why it's needed."
date: 2022-01-23T08:58:51+02:00
tags:
 - annileen
 - annileen-devlog
 - game-engine
 - graphics-programming
---

One of the first things to note once you pick BGFX as an abstraction to your graphics APIs, is that it has a [set of tools](https://bkaradzic.github.io/bgfx/tools.html) for asset conversion. And that is because it has importers for a few specific texture and mesh types and also a shader compiler that will generate shader for all the platforms supported, such as **OpenGL**, **DirectX** and **Metal**.

Since the main point of using BGFX is portability, not using these tools, especially the shader compiler, is stupid. Creating a proper asset management pipeline was necessary when porting from OpenGL because all the assets used before would have to go through this process.

So I thought of creating a structure of folders for the different types of assets, all of them which would have a *descriptor* file with import settings--something similar to what Unity does with its *metafiles*--and then have a tool to go asset by asset and *"import"* (another word I brought from Unity's pipeline, but ultimately means convert into the actual formats the engine will consume). Something like this:

```
 assets/
    cubemaps/
    fonts/
    models/
    shaders/
    textures/
```

And after importing the assets, it will create a folder `build_assets` with the final version of all the assets after going through BGFX tools. I picked python to write the asset tools because most of it would be dealing with paths, files, and invoking programs with several command-line instructions and it would be a huge time saver. The code for the asset tools can be [found here](https://github.com/CrociDB/annileen/tree/master/tools). A [section in the main README](https://github.com/CrociDB/annileen#asset-tools) of the project explains how to use the individual tools, but this article is meant to explain how they work and why they're there.

The asset-tools are only a small part of the whole Asset Manager. After it passes all the asset through the BGFX tools and gets the final version of them, it then creates a list of assets with their paths, in a way that it's easy to import inside the engine and in a way you only need to specify the name of the original file, and it will load the optimized file with the important information already specified in its descriptor. The asset list file looks like this:

```toml
[asset."blocks.png"]
path = ".\\build_assets\\textures\\blocks.dds"
type = "texture"

[asset."texture.png"]
path = ".\\build_assets\\textures\\texture.dds"
type = "texture"

[asset."bunny.obj"]
path = ".\\build_assets\\models\\bunny.obj"
type = "mesh"

[asset."lit_shadow.fs"]
path = ".\\build_assets\\shaders\\lit_shadow.fs"
type = "shader"

[asset."lit_shadow.vs"]
path = ".\\build_assets\\shaders\\lit_shadow.vs"
type = "shader"
``` 

Then loading a shader in annileen is like:

```cpp
auto shader = ServiceProvider::getAssetManager()->loadShader("lit_shadow"); // will load both fragment and vertex
```

It doesn't matter if you're using a Vulkan backend and the shader is compiled to SPIRV or a GLSL source for the OpenGL backend; you just need to specify the name of the file on the source `asset/` folder. Similarly, loading a texture is as simple as:

```cpp
auto texture = ServiceProvider::getAssetManager()->loadTexture("blocks.png");
```

The final texture format very unlikely will be PNG, and it will use the *descriptor* file for additional information. For example, the one for **blocks.png** is **blocks.toml** and its contents are, which I don't need to explain because it's self-explanatory:

```toml
mipmap = true
filter = "point"
gamma = "gamma"
```

Again, using the same philosophy as Unity engine, the *descriptors* are files with the exact same name and path as the asset, but with the `toml` extension.

***Quick note**: I picked `toml` for default data/setting/descriptor format in the engine because it's fairly easy to read by humans, especially because at first I wasn't thinking of having an editor in the engine.*

So, to recapitulate, the purposes of Annileen's asset manager are:

 - pass all the assets through the BGFX tools
 - compile the shaders for the target platform
 - offer an easy way to load assets in the game
 - manage resources in memory
   - make sure we have only one of each asset in memory and distribute references to it
   - unload unused assets
   - hot reload assets changed outside

In order to have the `asset_tools` working, we just have to run `$ python3 tools/asset_tools.py` on the root directory:

<script id="asciicast-oqYugvn3ckb6rs9RDg2ieQ1Zp" src="https://asciinema.org/a/oqYugvn3ckb6rs9RDg2ieQ1Zp.js" async></script>

# Texture and Mesh tools

As I mentioned in the first post, BGFX docs are *a little* precarious. There's a lot of information missing and it's incredibly difficult to actually understand how it works just by reading it. Not to blame anyone working on the project, it's an excellent open source project mantained by not many people, so it's fine. But just as an example, here's the description for one of the tools, the [**Geometry Compiler (geometryc)**](https://bkaradzic.github.io/bgfx/tools.html#geometry-compiler-geometryc):

> Converts Wavefront .obj, or glTF 2.0 mesh file to format optimal for using with bgfx.


It also says it supports `obj`, `gltf` and `glb` as input files and explains how to use its command line options, but no where you can find what the *"optimal format for using with bgfx"* is. The texture compiler, however, is a bit more detailed with the textures that BGFX will natively support, such as `dds`, `exr`, `hdr`, `ktx` and `png`.

Annileen uses the texture compiler (texturec) tool to compile texture into `dds` (Direct Draw Surface) and uses the BGFX texture importer, but not the mesh tool. It uses [*assimp*](https://github.com/assimp/assimp) for mesh loading and processing into its in-memory data structures. This means that the mesh tools in the **asset-tools** doesn't do anything right now but copying the mesh file into the `build_assets` folder.

# Shader tools

The BGFX shader language deserves an article on its own, so I won't go very deep in it, but it's basically GLSL with some macros and meta-data so it can *compile* into proper GLSL or HLSL. Annileen's shader tools will basically call the BGFX tool with the right parameters to get it compiled to the right platform. You can expect a devlog on it soon because I'm working on shader hot-reloading at the moment, and I plan to write a shader-variant system soon.

# Other tools

If you look into the [`tools/`](https://github.com/CrociDB/annileen/tree/master/tools) directory or check the [main docs](https://github.com/CrociDB/annileen#asset-tools) we have, you'll see it also has tools for fonts and cubemaps. Both of them will basicaly just copy the files into the final asset folder.

# Problems and Changes

I'm not really happy with the asset file structure. Except for shaders and textures, all the other asset types are not really pre-processed, so the files are just copied. That was because I didn't originally plan on implementing an in-engine editor for the engine, so the asset-tools were basically compiling the assets for *distribution*. But it changed and now we have an editor that might be able to edit assets in runtime. It won't edit the content of textures, meshes or shaders, but it will change the descriptors of these assets as well as composite assets such as cubemaps, materials, scenes, etc. With the current structure, I'd have to run the assets tools and copy all these assets to the `build_assets/` folder on every edit.

An option is keeping the `asset/` folder as the final asset folder, but putting the compiled shaders and textures into a `asset/build_assets/` sub-folder, so no need to keep copying the other assets. But also keep a build function that would create an optimal `build_assets/` folder for distribution. But these are only thoughts and plans. For now I'm satisfied enough with the way it currently works.

More [Annileen Devlogs](/tags/annileen-devlog).
