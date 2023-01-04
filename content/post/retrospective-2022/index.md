---
title: "Retrospective 2022"
description: "A retrospective of projects, life achievements and books of 2022"
date: 2023-01-03T12:33:51+02:00
draft: true
tags:
 - retrospective
---

For the last few years I've been writing a tweet in the end of the year with some of my life achievements, projects and professional changes. This year, based on other blogs I read, I decided to make a more complete post with even more content and include also other interesting things like some of the book I read and important concepts I learned this year.

# Projects

## Blog

This is, of course, one of the most important project and achievement of this year: I finally decided to write more to this blog. I've had blogs for over 15 years, but never really took it so seriously as of now. I read somewhere _"don't think to write, write to think"_ and it got to me so much I decided to write a lot more my projects and experiments. And it's been an amazing journey, I can feel how much writing has improved the way I think and learn about stuff.

My post about **doomoji** made it to the first page of Hacker News and got a lot of upvotes on the `/r/programming` subreddit.

![I'm on the front page of Hacker News!](images/doomoji-hackernews.jpg)

That all gave me more confidence to write more as a tool to learn more and get feedback for silly experiments like that one. I learned so much from the comments and nobody was hostile or pointing problems just for the sake of annoying, as I sadly expected.

## wordlos

Early 2022 was the year that everyone started posting those weird green/yellow/gray boxes to twitter and it took me some time to figure out it was about a new _cult_ word game: *wordle*. Didn't take long after for me to get addicted to it as well. So addicted indeed I also started playing other of the countless variations, such as the [Brazilian Portuguese](https://term.ooo/) version of it.

After having written [retro2048](https://crocidb.github.io/retro2048/), a port of the other [new classic 2048 game to the bootsector](https://crocidb.com/post/bootsector-game/) and DOS, I wanted to do more text-mode assembly games and wordle seemed like a perfect match for my needs. So I took a few weeks to work on it. The result is a perfectly fun wordle version of it that runs on any DOS system.

![wordlos](https://github.com/CrociDB/wordlos/raw/main/screenshot/wordlos1.gif)

This is was also the first time I got a pull request on an open-source project I ever wrote. [Paweł Łukasik](https://github.com/pawlos) wrote a fix for an issue with repeating letters and submitted it. It was such a cool moment to see people genuinely interested in a project I made to write a fix and submit a PR.

I didn't write anything about this project around here yet, but I want to at some point, at least about how I made to port it to be played online on itch.io, using a wasm version of DosBox and taking the output framebuffer to be rendered with ThreeJS with a CRT post-processing effect, looking like this:

![wordlos for web](images/wordlos-web.png)

## Space Lord X

For the first time in my life I finished an entry for Ludum Dare. So many times I started, but left at some point because of my extreme incapacity of making a proper game design.

This time I decided to pretty much clone a game for Atari 2600 that I used to play a lot as a kid, The [Space Master X-7](https://www.youtube.com/watch?v=ia4rGNuIh1g&t=10s). Thought that would be a cool experiment to implement in the WASM-4 fantasy console, so this was the result:

![Space Lord X](https://img.itch.zone/aW1hZ2UvMTcyOTI2NS8xMDE4NTE1NS5naWY=/original/hWJQ0Y.gif)

It can be played on [itch.io](https://crocidb.itch.io/spacelord-x). I stll haven't written a post about this project here because I'm still working on improvements post-compo such as improving the player movement, boss behaviours and adding sound. I think input issues and the lack of audio really brought the game rating down, but I was very happy with the results.

![Space Lord X's Ludum Dare Result](images/space-lord-x-results.png)

## Photography Website Generator

For some time I've been disappointed with all the photo platoforms, nobody uses Flickr anymore; Instagram became another tiktok and doesn't distribute your photos anymore; pretty much all the other platforms are now all about NFTs. I considered many ways I could create a photo website and ultimately decided to create a static website generator: flingern.

It's still on an early stage of development, but I can already create basic websites with it:

[![Photo Website](images/photography-website.png)](http://photos.crocidb.com/city.html)

It's already capable of:

 - importing photos and set the quality/max size for both the thumbnails and display photo
 - generating pages from Markdown
 - website structure is defined using a simple yaml file

 Next features I want to support before calling it a proper version are:
 
  - display photo exif information
  - photo tags, so it's possible to see photos by tags and not only by pages
  - improve the layout of the theme
  - improve the experience using the command line client of _flingern_

To be honest, dealing with HTML/CSS has been one of my worst issues here. I have worked with it on the late 2000's, but haven't really kept up to date with it, so it's always a pain for me to make page layouts. If you're into that and feel like could be a good hand, please don't hesitate to contribute to: https://github.com/CrociDB/flingern.

# Photography

By the end of 2021 I was introduced to film photography and throughout the whole 2022 I pretty much only shot film. I wrote a [post about it here](https://crocidb.com/post/one-year-film-photography/) and also created a page to display my current [film camera collection](https://crocidb.com/cameras/), as it's became a very important hobby for me.

![Some photos I took this last year](https://crocidb.com/post/one-year-film-photography/images/some-photos.jpg)

# Wacken Open Air - Music Festival

This year I went to my first music festival ever, and it wasn't just any festival, it was Wacken Open Air, the holy land for Metal fans.

This was such a unique experience. And I say that not necessarily in the best possible way, because part of it was really awful for me. I'm not used camping or being in a place with so many people for such a long time like that. I love metal concerts and even though I'm going to many of them during these years, being in there was completely overwhelming for me. As if it wasn't too much already, I was also a bit sick with a sore aching throat and coughing and completely deaf from one ear, as I'd just come back from Brazil the previous day the pressure change during landing messed my ears.

Constant bathroom lines and temperature variations (it was over 30 degrees during the day but got to 5 at night) on top of that made part of my experience pretty shitty. But I'm really I was with friends that were very supporting and even if I might have ruined part of their experience, they were acknologing and helping, and that's because of them that the good part of the whole experience was actually fantastic and I also got a ticket for 2023!

To my friends Thiago and Jon and my girlfriend Fredi: I am deeply sorry for ruining part of it, but thank you all very much for being understanding and making this much better than I could expect.

![Wacken Crew](images/wacken-crew.jpg)

# German Learning

I've been living in Germany for almost three years at this point and my German is still pretty basic. I did A1/A2 German school by 2020, but after that I dropped the courses and was mostly doing duolingo and try to _learn by living_. It works when you are exposed to the language every time of every day, but I am not, even though I live in Germany. The official language in the office is English, my closest friends here are also not from Germany, so English speaking pretty much all the time.

There are, of course, situations where I'm in a place with pretty much only German speakers, and I have to do it. These situations have been a little bit more common this year and I feel like my German has improved a bit because of this. I can understand a lot more than I could last year. Sometimes I even identify the words, but don't understand what they mean.

For 2023 I'll try to be more active and try to read more and try to talk a lot more. I wonder if I should get a proper private teacher for some sessions a month to help me going.

# Professional

By the end of the year I moved positions within the company. I became now an Audio Programmer. Even though I don't have much experience with it, the Audio team embraced me and are very happy to have me learning all I can abut it.

 - moved to being an audio programmer 
 - working with an in-house engine 
 - programming c++

# Books/Tech
 - realm of racket

# Books/Graphic Novels 
 - yellow cab
 - polina
 - la obsolescência programada de nuestros sentimientos
 - exit wounds
 - ballad for Sophie
 - call me Nathan

# Books/Fictions
 - 