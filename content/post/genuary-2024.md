---
title: "My Genuary 2024 Entries"
description: "Genuary is a generative art event with daily prompts in January."
date: 2024-04-27T11:57:44+02:00
draft: false
tags:
 - creative-coding
 - generative-art
 - art
 - programming
---

Genuary is a generative art event that happens in January, where you have a prompt a day and have to implement some cool art based on them. Conceptually, it's situated in a place somewhere in between the demoscene and NFT Art (probably closer to the latter), but with very interesting prompts nonetheless.

I entered the party a bit later, by the end of January, and did as many prompts as I felt like, which was 9, out of 31. Not too bad for a first time. And I had a lot of fun. I decided to use whatever language/framework I felt like for each individual prompt, but ended up using mostly **p5.js** (Javascript) and **TIC-80** (Lua).

All my entries are [in this repo](https://github.com/CrociDB/genuary/tree/main/2024), but here I am commenting each of of them:

## 1. Particles

![Particles](https://github.com/CrociDB/genuary/raw/main/2024/1/particles.gif)

I was actually working on an entry for [Lovebyte Party 2024](https://lovebyte.party/) in **TIC-80**, so I decided to start this in that platform, so I could improve my skills in it. I'm not particularly proud of this one, but I do think it looks kinda cool.

## 2. No Palettes

![No Palettes](https://github.com/CrociDB/genuary/raw/main/2024/2/no-palettes.gif)

It's such a broad prompt, but I guess the main point is to use dynamic colors, so I fired up **p5.js**, which I hadn't used in years. It turned out to be one of my favorites, and to be honest this gif doesn't show it properly, but you can see it running in real time here: [No Palettes](https://editor.p5js.org/CrociDB/full/0q7Ud4nJW)

## 3. Droste Effect

![Droste Effect](https://github.com/CrociDB/genuary/raw/main/2024/3/droste.gif)

Droste Effect is when you are recording a live feed of the recording itself, creating an infinite-tunnel-like effect. I had an idea it would be good to do it in 3D, so I went for a shader in **Shadertoy**. It looked better in my mind, but I was taking too long to implement it, so I just finished it like this. It's the one I like the least, and I only kept it because it looks interesting on a big screen running at 165Hz. If you can run it, I suggest trying it [here](https://www.shadertoy.com/view/4cSXRR).

## 4. Pixel

![Pixel](https://github.com/CrociDB/genuary/raw/main/2024/4/pixel.gif)

Simple one, it shows the individual colors in a pixel and zooms out to show an image of Mario. I like the effect. The live version is [here](https://editor.p5js.org/CrociDB/full/LixZY5gnN).

## 5. Vera Molnár

![Vera Molnár](https://github.com/CrociDB/genuary/raw/main/2024/5/molnar.gif)

I didn't know the work of this artist, and I looks like her work is often referenced in NFT Art. I picked her work [**(Des)Ordres**](https://dam.org/museum/artists_ui/artists/molnar-vera/des-ordres/) and basically replicated it in **p5.js**, with some smooth animation changing the lines. I like the results.

## 6. Screensaver

![Screensaver](https://github.com/CrociDB/genuary/raw/main/2024/6/screensaver.gif)

Inspired by old Windows 98 screensavers, I came up with this simple star bouncing on the edges of the screen in **TIC-80**. Paired with the default CRT screen shader in the platform, it looks cool.

## 7. Loading

![Loading](https://github.com/CrociDB/genuary/raw/main/2024/7/loading.gif)

I'm no UI/UX expert, so implemented a loading screen inspired by one of the best video games ever, [Castlevania: Symphony of the Night](https://i.redd.it/help-symphony-of-the-nights-on-psp-keeps-crashing-after-the-v0-k5ruruuhiphb1.jpg?width=1920&format=pjpg&auto=webp&s=838df4bc897e49e0d585af4ccad2743bdc15accb). The wayvy motion is inspired by the loading screen in the game and the "ghosting" inspired by the famous Alucard ghosting. I love this.

## 8. Chaotic System

![Chaotic System](https://github.com/CrociDB/genuary/raw/main/2024/8/chaos.gif)

I tried to do something similar to the plotting of the [Lorentz System](https://en.wikipedia.org/wiki/Lorenz_system), the classic "Chaos Theory" graph, but a lot simpler.

## 9. ASCII

![ASCII](https://github.com/CrociDB/genuary/raw/main/2024/9/asciicamera.gif)

I'm rendering the webcam stream with ASCII characters based on luminosity, but also using the colors from the stream. Also added some "scanlines" effect. I think it looks nice. You can try it [here](https://editor.p5js.org/CrociDB/full/7rPr8bIzJ).

---

That was a fun experience. I want to do it next year again, maybe try to do daily like the challenge proposes.