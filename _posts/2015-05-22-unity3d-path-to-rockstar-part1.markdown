---
layout: post
title: The path to be a Unity3D Rockstar - Part 1
description: "First part of a small series of tips on how to be a Rockstar on Unity3D"
date: 2015-05-22 16:0:00
categories: articles
image:
  feature: "http://crocidb.com/assets/img/unity1-feature.jpg"
---

I've been working with Unity3D full time for the last two years now. I'm still learning, but I would like to share some of the things I learned working with rockstars, and maybe that can lead you to the right path.

I remember the first time I used Unity3D. I really didn't like it. It felt like a maker. I'd import 3d models and attach scripts I found online and there was my game. That was too limited. But that's because I was used to code everything and taught that makers are always limited. That isn't true. After using Unity for several projects, I learned how great this tool is. Sometimes, if it sounds too limited to you, maybe you just don't know the engine.

[Thiago](https://www.facebook.com/thiago.cava), a friend of mine who always loved the engine even when I didn't, asked me some guidance on it. I sent him a list of fundamental topics and points I have to deal with everyday, so I decided to make more general and detailed here.

I divided this article in two parts, so stay tunned for the next one.

### Know the language

I'm a programmer and this guide is focused on programmers. So there's no better way to start this list than with the most used tool for us, the language. I've seen people using C# over the UnityScript and Boo for game development, so I'll focus on it.

Understanding the language you're using deeply is perhaps the most important thing on this list. You need to know the basic and exclusive things of the language, such as Actions, Delegates, Lambda Functions, how OOP works, Generics, etc. C# is a very powerful language (way better than Java :O) full of interesting things.

I love Javascript, but don't get fooled, UnityScript isn't Javascript. However, if you're using it, pay attention to its exclusive features and master it.

### GameObjects

Everything is a GameObject, so know it. The main topics on it are:

 - Instantiate objects
 - Destroy objects
 - Find objects
 - Adding components to objects
 - Getting components on objects
 - Interacting with components
 - Understand the lifecycle of a MonoBehaviour
 - Understand the Transform object

It may seem basic, but if you know them, you can have full control over a GameObject.

### Custom Editor Tools

It's very likely that you'll take a lot of your programming time to create tools to make some tasks easier. If you don't do that, you should.

Unity editor is very flexible, you can do a lot with it. Create custom windows, custom menus, cusom inspector, etc. Using this the right way can make more organic and smooth to create new features and tweak current features in your game.

### Scriptable Objects

Another thing you'll need is a way to store data. Let's say you need to store the settings for each gun your game has. You can use XML, Json, Yaml and other markup languages, but Unity provides the [ScriptableObjects](http://docs.unity3d.com/Manual/class-ScriptableObject.html).

Those are mostly like a markup language, but it's editable by the editor. So you can easily change each object, create new and delete. Also, you'll want to create tools to make it even easier. Dealing with ScriptableObjects in Unity is just like dealing with any other object.



* * *

Headline photo was found [on Picography](http://picography.co/photos/long-road-ahead/).

