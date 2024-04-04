---
title: "Annileen Devlog #0 - Introduction"
description: "Annileen is a 3D game engine created for study puroposes and I'm starting a devlog series on it."
date: 2021-10-26T08:58:51+02:00
tags:
 - annileen
 - annileen-devlog
 - game-engine
 - graphics-programming
---

Back in 2018, I was writing shaders more often than ever and an interest in graphics programming sparked. Everybody was talking about Vulkan or DirectX12, but I didn't even know OpenGL, or the difference between all of them. On my first years of game programing, I remember the always-recommended [NeHe OpenGL Tutorials](https://nehe.gamedev.net/), that I could never do because I couldn't understand much English.

That was definitely time to learn some OpenGL, and I was presented with the more cohesive and modern [LearnOpenGL](https://learnopengl.com/), which I couldn't recommend more, with the acessible language and lot of content, it will not only teach you OpenGL but several of the most important techniques to build a fully featured 3d renderer. I started creating a Minecraft clone, because voxels were also a fascinating subject to me.

I called it **clonecraft** and not long after I had something like this:
{{< tweet id=1075890855217688578 user=crocidb >}}

That was really satisfying that basically only following **LearnOpenGL** and reading some random texts online on how voxels work I was able to implement that. Even though it was supposed to be a "simple minecraft clone", I was implementing it in an abstract way, similar to how Unity engine works in my mind, except not following the component model. So I had classes for Transform, GameObject, Scene, Material, Mesh, Shader, etc. That was almost feeling like an engine.

In 2019 I moved out of Brazil and the project was sitting there for a while, untouched. That was the year, however, that I started implementing a [raytracer](https://github.com/CrociDB/pathtracer), following the really amazing [Ray Tracing in One Weekend](https://raytracing.github.io/books/RayTracingInOneWeekend.html), by [Peter Shirley](https://twitter.com/peter_shirley). That year was also when I started [writing shaders on Shadertoy](https://www.shadertoy.com/user/crocidb) and learned a lot about [raymarching signed-distance functions](https://www.iquilezles.org/www/articles/distfunctions/distfunctions.htm) and how to use [constructive solid geometry](https://en.wikipedia.org/wiki/Constructive_solid_geometry) to model objects and worlds. So even though I wasn't very active working on my _minecraft clone_, I was still studying a lot of graphics programming. 

In 2020, talking to my good friend [Teo Dutra](https://teodutra.com/), currently a graphics engineer at Unity Engine, I decided to write a proper toy engine so I could implement graphics techniques such as a PBR model, forward and deferred renderers, a global illumination system, and also engine basic stuff like a scene graph, asset management, etc and finally have a game made on it. I was very excited because he said he would also contribute to the project. That's how **Annileen** was born.

## Annileen Engine

First, before anybody asks, the name is based on a very important and inspiring character from the Star Wars universe. [Annileen Calwel](https://starwars.fandom.com/wiki/Annileen_Calwell) is a woman who lives in Tatooine and owns a bar/store very close to where **Obi-Wan Kenobi** lives after he delivers Luke to that adorable couple. It's from the **Kenobi** novel, which is currently considered **Legends** and not cannon, at least until the new [Disney+ Kenobi series](https://en.wikipedia.org/wiki/Obi-Wan_Kenobi_(TV_series)) is out. I picked it because I like the character and I think the name sounds really good.

One thing was right: we couldn't keep using OpenGL because it was being deprecated by some systems and with the tendency of all the graphic APIs to get lower and lower level, that sounded wrong. Vulkan was the best option, but I had written a helloworld not long before and I was very overwhelmed with having to learn so many low-level concepts before getting to be able to implement a lighting model. Then I started looking for other alternatives and I stumbled upon [BGFX](https://github.com/bkaradzic/bgfx), a _"cross-platform, graphics API agnostic, 'Bring Your Own Engine/Framework' style rendering library"_, or a graphics API abstraction that would help my project run on both OpenGl, DirectX12, Vulkan, Metal and WebGL with the same implementation.

Of course there was discussions that it might not be a good idea, because the purpose would be to learn and implement the APIs ourselves, but I wanted to have a quicker start, being able to start implementing some actual graphic techniques before having to understand all the quirks from a low-level API. And plus, BGFX was used in the rewrite of Minecraft that Microsoft did after buying Mojang, so that inspired me and I started porting **clonecraft** to use BGFX instead of OpenGL. It took me some time to do the port, especially BGFX doens't have very detailed docs. However, there are many examples, and most of port was based on reading the examples.

After the porting, we kept working on it. Teo implemented a shadowmap feature, imported **imgui** and started creating an editor, we invested sometime in making a build system with **premake5** that would run in both Windows and Linux (especially to use Travis as CI), a log system, an asset/resource manager, font rendering, model loading and etc.

![Annileen Engine](https://github.com/CrociDB/annileen/raw/master/screenshot/annileen.jpg)

I work on the project on my free time and I try to conciliate with other personal projects too, so sometimes it would take months to get something done and I might get other months not working on it. But it's slowly going somewhere. I decided to write devlogs in here so it would be one more source of motivation for this study project.

There's still a lot to talk about the features we currently have and the ones we're planning on implementing soon. I'm very open for discussion and, of course, contributions, since the project is completely opensource and is on [GitHub](https://github.com/CrociDB/annileen).

More [Annileen Devlogs](/tags/annileen-devlog).
